# Simple Time-Locked Vault (Solidity + Foundry + Celo Alfajores)

## 📑 Índice

1. [🛠️ 1. Instalar Foundry](#🛠️-1-instal-foundry)
2. [📦 2. Instalar Celo CLI](#📦-2-instal-celo-cli)
3. [⚙️ 3. Configurar Celo CLI](#⚙️-3-configurar-celo-cli)
4. [👛 4. Setup de billetera](#👛-4-setup-de-billetera)
5. [💰 5. Fondeo con el faucet de Alfajores](#💰-5-fondeo-con-el-faucet-de-alfajores)
6. [🚀 6. Inicializar proyecto](#🚀-6-inicializar-proyecto)

## 🛠 Prerrequisitos

Antes de empezar, asegúrate de tener:

> **¿Por qué Foundry?**
> Foundry se elige en esta guía por su velocidad impresionante, compatibilidad nativa con Solidity y mínima configuración. Es la opción preferida para quienes buscan una herramienta ligera y eficiente frente a frameworks como Hardhat o Truffle.

* Un terminal (Linux/macOS preferido; **en Windows** usa WSL 2 o Git Bash. En PowerShell, verifica que `forge`, `cast` y `celocli` estén en tu `%PATH%`).
* **Node.js versión 18 o superior** para poder instalar la CLI de Celo sin problemas.
* Conocimientos básicos de Solidity y contratos inteligentes.
* Navegador web para acceder al faucet del testnet de Celo.

---

## 📋 Descripción general del contrato

#### Contexto
El TimeLockVaultFactory es un contrato factory que permite a los usuarios crear bóvedas con bloqueo temporal tanto para CELO nativo como para tokens ERC-20. Cada bóveda bloquea fondos hasta una hora de desbloqueo especificada.

**Características principales:**
* Crear bóvedas con bloqueo temporal para CELO y tokens ERC-20
* Mecanismo de retiro seguro con validación de tiempo
* Patrón factory para gestión de múltiples bóvedas
* Control de acceso y medidas de seguridad

---

## 🚀 Descripción del flujo de trabajo

1. [Instalar Foundry](#1-instalar-foundry)
2. [Instalar Celo CLI](#2-instalar-celo-cli)
3. [Configurar Celo CLI](#3-configurar-celo-cli)
4. [Crear una billetera](#4-setup-de-billetera)
5. [Obtener fondos](#5-fondeo-con-el-faucet-de-alfajores)
6. [Inicializar el proyecto](#6-inicializar-proyecto)
7. [Escribir tu contrato](#7-escribir-el-contrato)
8. [Compilar el contrato](#8-compilar-el-contrato)
9. [Probar el contrato](#9-probar-el-contrato)
10. [Desplegar en Alfajores](#10-despliegue)
11. [Interactuar con el contrato](#11-interactuar-con-el-contrato)

---

### 🛠️ 1. Instalar Foundry

Foundry es un toolkit para compilar, testear y desplegar contratos inteligentes.

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup               # Versión estable (recomendada)
# O para la versión más reciente:
# foundryup -i nightly
```

> **Tip:** una vez instalado, confirma que Foundry quedó operativo comprobando la versión desde tu terminal.

---

### 📦 2. Instalar Celo CLI

Instala la CLI de Celo para manejar cuentas e interactuar con la red.

```bash
npm install -g @celo/celocli
```

---

### ⚙️ 3. Configurar Celo CLI

Apunta la CLI al testnet Alfajores de Celo.

> **¿Qué es Alfajores?**
> Es el testnet público de Celo: aquí pruebas con tokens sin valor real antes de ir a Mainnet.

```bash
celocli config:set --node=https://alfajores-forno.celo-testnet.org
celocli config:get
```

---

### 👛 4. Setup de billetera

En Web3, tu billetera es tu dirección pública (para recibir fondos) y tu llave privada (para firmar transacciones). **Nunca compartas tu llave privada ni la subas a repos públicos.** Considera usar herramientas como `direnv` o un gestor de secretos para evitar exponerla en el historial de tu shell.

```bash
celocli account:new
```

Copia y guarda con cuidado la dirección y la llave privada.

Luego exporta variables de entorno:

```bash
export YOUR_PRIVATE_KEY=<tu_llave_privada>
export YOUR_ADDRESS=<tu_dirección>
```

Importa la llave en Foundry:

```bash
cast wallet import wallet-hello-celo --private-key $YOUR_PRIVATE_KEY
# Se te pedirá crear una contraseña para el keystore
```

Crea un archivo `.env` para no repetir estos valores:

```bash
# .env
CELO_ACCOUNT_ADDRESS=$YOUR_ADDRESS
CELO_NODE_URL=https://alfajores-forno.celo-testnet.org
```

---

### 💰 5. Fondeo con el faucet de Alfajores

Necesitas CELO (y opcionalmente cUSD) para desplegar y usar contratos. En testnet es gratis.

1. Ve a [https://alfajores.celo.org/faucet](https://alfajores.celo.org/faucet)
2. Pega tu dirección y solicita CELO (y cUSD si quieres).
3. Verifica tu saldo:

   ```bash
   celocli account:balance $CELO_ACCOUNT_ADDRESS
   ```

---

### 🚀 6. Inicializar proyecto

Crea un nuevo proyecto de Foundry para tu contrato de bóveda con bloqueo temporal.

```bash
forge init time-locked-vault
cd time-locked-vault
```

> **Tip:** Foundry crea automáticamente la estructura de directorios y archivos necesarios para tu proyecto.

---

### ✍️ 7. Escribir tu contrato

A continuación desglosamos los componentes del contrato `TimeLockVaultFactory`.

---

### 📄 7.1. Licencia SPDX e Interfaz

#### Contexto
Comienza con el identificador de licencia y define la interfaz ERC-20 para las interacciones con tokens.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IERC20 {
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
    function transfer(address to, uint256 amount) external returns (bool);
}
```

---

### 🗂 7.2. Variables de estado y estructuras

#### Contexto
Define las variables de estado y las estructuras de datos necesarias para almacenar la información de las bóvedas.

```solidity
contract TimeLockVaultFactory {
    /// @notice El deployer (podría usarse para acciones de admin)
    address public immutable owner;

    /// @notice ID de bóveda auto-incrementable
    uint256 private nextVaultId;

    /// @notice Una bóveda con bloqueo temporal (CELO o ERC-20)
    struct Vault {
        address creator;
        address token;       // cero = CELO, sino contrato ERC-20
        uint256 amount;      // cantidad de CELO o tokens bloqueados
        uint256 unlockTime;  // timestamp cuando se permite el retiro
        bool withdrawn;
    }

    /// @dev mapping de `vaultId` a `Vault`
    mapping(uint256 => Vault) public vaults;

    /// @dev mapping de `creator` a lista de `vaultIds`
    mapping(address => uint256[]) public userVaults;
}
```

> **⚠️ Importante:** La variable `owner` es inmutable para evitar cambios después del despliegue, garantizando la seguridad del contrato.

---

### 🔧 7.3. Constructor

#### Contexto
Inicializa el contrato con un parámetro opcional `_owner` para flexibilidad en el despliegue.

```solidity
/// @param _owner Override opcional (pasa `address(0)` para usar `msg.sender`)
constructor(address _owner) {
    owner = _owner == address(0) ? msg.sender : _owner;
}
```

> **📝 Nota:** Pasar `address(0)` como `_owner` utiliza `msg.sender` como dueño, proporcionando flexibilidad en el despliegue.

---

### 🛠 7.4. Función helper interna

#### Contexto
Crea una función privada helper para registrar la bóveda y mantener la consistencia en las creaciones.

```solidity
/// @dev helper interno para registrar la bóveda
function _storeVault(
    address creator,
    address token,
    uint256 amount,
    uint256 _unlockTime
) private returns (uint256 vaultId) {
    vaultId = nextVaultId++;
    vaults[vaultId] = Vault({
        creator:   creator,
        token:     token,
        amount:    amount,
        unlockTime:_unlockTime,
        withdrawn: false
    });
    userVaults[creator].push(vaultId);
}
```

> **💡 Tip:** Usar una función helper privada garantiza lógica consistente y reduce duplicación de código.

---

### 🔒 7.5. Creación de bóveda ERC-20

#### Contexto
Crear bóvedas con bloqueo temporal para tokens ERC-20 con verificaciones de aprobación apropiadas.

```solidity
    /// @notice Bloquear ERC-20 `token` hasta `_unlockTime`
    /// @dev el llamador debe `approve` este contrato por al menos `_amount` primero
    function createVaultERC20(address token, uint256 amount, uint256 _unlockTime)
        external
        returns (uint256 vaultId)
    {
        require(amount > 0, "La cantidad debe ser >0");
        require(_unlockTime > block.timestamp, "El tiempo de desbloqueo debe ser en el futuro");
        require(token != address(0), "Dirección de token inválida");

        // transferir tokens
        bool ok = IERC20(token).transferFrom(msg.sender, address(this), amount);
        require(ok, "Falló la transferencia de tokens");

        vaultId = _storeVault(msg.sender, token, amount, _unlockTime);
    }
```

> **⚠️ Advertencia:** Los usuarios deben aprobar que el contrato gaste sus tokens antes de llamar esta función. Siempre verifica el estado de aprobación en tu frontend.

---
### 💸 7.6. Función de retiro

#### Contexto
Implementa un mecanismo de retiro seguro con múltiples comprobaciones de validación para prevenir accesos no autorizados y retiros duplicados.

```solidity
    /// @notice Withdraw funds after unlock time
    function withdraw(uint256 vaultId) external {
        Vault storage v = vaults[vaultId];

        require(v.creator != address(0), "Vault does not exist");
        require(msg.sender == v.creator, "Not vault creator");
        require(block.timestamp >= v.unlockTime, "Too early");
        require(!v.withdrawn, "Already withdrawn");

        v.withdrawn = true;
        if (v.token == address(0)) {
            // CELO
            payable(v.creator).transfer(v.amount);
        } else {
            // ERC-20
            bool ok = IERC20(v.token).transfer(v.creator, v.amount);
            require(ok, "Token transfer failed");
        }
    }
```

---

### 🛠️ 8. Compilar el contrato

#### Contexto
Compila tu contrato para verificar que no hay errores de sintaxis.

```bash
forge build
```

> **💡 Tip:** Foundry mostrará errores con detalles para facilitar la corrección.

---

### 🧪 9. Pruebas unitarias

#### Contexto
Ejecuta pruebas para verificar la funcionalidad del contrato.

### 📋 Prerrequisitos de pruebas

* **Foundry** instalado
* Comprensión básica de pruebas en Solidity
* Familiaridad con el framework de pruebas de Foundry
* Entendimiento de conceptos de bóveda con bloqueo temporal
* Conocimiento de estándares de tokens ERC-20

> **💡 Tip:** Foundry busca archivos de prueba en el directorio `test/` con extensión `.t.sol`.

---

### 🏗️ Crear estructura de pruebas

#### Contexto
Configura la estructura de directorios para tus archivos de prueba siguiendo las convenciones de Foundry.

```bash
mkdir -p time-lock-vault/test
touch time-lock-vault/test/TimeLockVaultFactory.t.sol
```

> **💡 Tip:** Foundry recopila automáticamente archivos de prueba en `test/`.

---

### ⚙️ Inicializar archivo de prueba

#### Contexto
Crea la estructura básica del archivo de prueba con las importaciones necesarias y la configuración del contrato.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/TimeLockVaultFactory.sol";

contract TimeLockVaultFactoryTest is Test {
    TimeLockVaultFactory public factory;
    address public bob = address(0x2);

    function setUp() public {
        factory = new TimeLockVaultFactory(address(0));
        vm.deal(bob, 10 ether);
        vm.deal(address(this), 10 ether);
    }

    receive() external payable {}
}
```

---

### 1. Prueba de flujo feliz de bóveda CELO

#### Contexto
Verifica el ciclo completo de una bóveda de CELO desde su creación hasta el retiro.

```solidity
function test_CeloVault_HappyPath() public {
    // Setup: enviar 1 ether con _unlockTime = block.timestamp + 1 days
    uint256 lockAmount = 1 ether;
    uint256 unlockTime = block.timestamp + 1 days;
    
    uint256 vaultId = factory.createVaultCelo{value: lockAmount}(unlockTime);

    // Verificar creación
    (address creator, address token, uint256 amount, uint256 vaultUnlockTime, bool withdrawn) = factory.vaults(vaultId);
    assertEq(creator, address(this));
    assertEq(token, address(0));
    assertEq(amount, lockAmount);
    assertEq(vaultUnlockTime, unlockTime);
    assertEq(withdrawn, false);

    // Avanzar tiempo
    vm.warp(block.timestamp + 1 days + 1);

    // Registrar balance antes de retiro
    uint256 balanceBefore = address(this).balance;

    // Retirar
    factory.withdraw(vaultId);

    // Verificar retiro
    (,, , , withdrawn) = factory.vaults(vaultId);
    assertEq(withdrawn, true);

    // Verificar recepción de fondos
    uint256 balanceAfter = address(this).balance;
    assertEq(balanceAfter, balanceBefore + lockAmount);
}
```

---

### 2. Prueba de flujo feliz de bóveda ERC-20

#### Contexto
Verifica la creación y retiro de una bóveda ERC-20 con un token mock.

```solidity
function test_ERC20Vault_HappyPath() public {
    MockERC20 token = new MockERC20();
    uint256 lockAmount = 100 ether;
    uint256 unlockTime = block.timestamp + 1 days;
    
    // Approve tokens
    token.approve(address(factory), lockAmount);
    
    // Crear bóveda
    uint256 vaultId = factory.createVaultERC20(address(token), lockAmount, unlockTime);
    
    // Verificar creación
    (address creator, address vaultToken, uint256 amount,, bool withdrawn) = factory.vaults(vaultId);
    assertEq(creator, address(this));
    assertEq(vaultToken, address(token));
    assertEq(amount, lockAmount);
    assertEq(withdrawn, false);
    
    // Avanzar tiempo y retirar
    vm.warp(block.timestamp + 1 days + 1);
    factory.withdraw(vaultId);
    
    // Verificar retiro
    (,, , , withdrawn) = factory.vaults(vaultId);
    assertEq(withdrawn, true);
    assertEq(token.balanceOf(address(this)), 900 ether);
}
```

---

### 3. Casos de falla y edge cases

#### Contexto
Prueba escenarios de falla para asegurar un comportamiento correcto bajo condiciones inválidas.

```solidity
function test_CreateVaultWithZeroAmount() public {
    uint256 unlockTime = block.timestamp + 1 days;
    
    vm.expectRevert("No CELO sent");
    factory.createVaultCelo{value: 0}(unlockTime);
}

function test_CreateVaultWithPastUnlockTime() public {
    uint256 pastTime = block.timestamp - 1 days;
    
    vm.expectRevert("Unlock time must be in future");
    factory.createVaultCelo{value: 1 ether}(pastTime);
}

function test_WithdrawTooEarly() public {
    uint256 lockAmount = 1 ether;
```

---

### 🚢 10. Desplegar en Alfajores

#### Contexto
Configura variables de entorno y despliega tu contrato.

```bash
export PRIVATE_KEY=$YOUR_PRIVATE_KEY
export RPC_URL=https://alfajores-forno.celo-testnet.org
forge script script/Deploy.s.sol:DeployScript --rpc-url $RPC_URL --broadcast --verify
```

> **⚠️ Importante:** Asegúrate de tener suficientes CELO en tu cuenta antes de desplegar.

---

### 🔗 11. Interactuar con el contrato

#### Contexto
Utiliza `cast` para interactuar con las funciones del contrato desplegado.

```bash
# Crear bóveda CELO
cast send <CONTRACT_ADDRESS> "createVaultCelo(uint256)" <UNLOCK_TIMESTAMP> --value 0.1ether --private-key $PRIVATE_KEY

# Verificar bóveda
cast call <CONTRACT_ADDRESS> "vaults(uint256)" <VAULT_ID>

# Retirar fondos
cast send <CONTRACT_ADDRESS> "withdraw(uint256)" <VAULT_ID> --private-key $PRIVATE_KEY
```

> **💡 Tip:** Sustituye `<CONTRACT_ADDRESS>`, `<VAULT_ID>` y `<UNLOCK_TIMESTAMP>` con valores reales.

---

**�� ¡Felicitaciones!** Has construido exitosamente un contrato inteligente de bóveda con bloqueo temporal seguro. Los siguientes pasos son pruebas exhaustivas, auditoría de seguridad e integración con tu aplicación.