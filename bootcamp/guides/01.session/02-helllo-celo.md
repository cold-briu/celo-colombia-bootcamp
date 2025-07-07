# 📝 Guía de HelloCelo

## 🛠 Prerrequisitos

Antes de empezar, asegúrate de tener:

> **¿Por qué Foundry?**
> Foundry se elige en esta guía por su velocidad impresionante, compatibilidad nativa con Solidity y mínima configuración. Es la opción preferida para quienes buscan una herramienta ligera y eficiente frente a frameworks como Hardhat o Truffle.

* Un terminal (Linux/macOS preferido; **en Windows** usa WSL 2 o Git Bash. En PowerShell, verifica que `forge`, `cast` y `celocli` estén en tu `%PATH%`).
* **Node.js versión 18 o superior** para poder instalar la CLI de Celo sin problemas.
* Conocimientos básicos de Solidity y contratos inteligentes.
* Navegador web para acceder al faucet del testnet de Celo.

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

### 1. Instalar Foundry

Foundry es un toolkit para compilar, testear y desplegar contratos inteligentes.

```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup               # Versión estable (recomendada)
# O para la versión más reciente:
# foundryup -i nightly
```

> **Tip:** una vez instalado, confirma que Foundry quedó operativo comprobando la versión desde tu terminal.

---

### 2. Instalar Celo CLI

Instala la CLI de Celo para manejar cuentas e interactuar con la red.

```bash
npm install -g @celo/celocli
```

---

### 3. Configurar Celo CLI

Apunta la CLI al testnet Alfajores de Celo.

> **¿Qué es Alfajores?**
> Es el testnet público de Celo: aquí pruebas con tokens sin valor real antes de ir a Mainnet.

```bash
celocli config:set --node=https://alfajores-forno.celo-testnet.org
celocli config:get
```

---

### 4. Setup de billetera

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

### 5. Fondeo con el faucet de Alfajores

Necesitas CELO (y opcionalmente cUSD) para desplegar y usar contratos. En testnet es gratis.

1. Ve a [https://alfajores.celo.org/faucet](https://alfajores.celo.org/faucet)
2. Pega tu dirección y solicita CELO (y cUSD si quieres).
3. Verifica tu saldo:

   ```bash
   celocli account:balance $CELO_ACCOUNT_ADDRESS
   ```

---

### 6. Inicializar proyecto

Crea la estructura base de Foundry:

```bash
forge init hello-celo
cd hello-celo
rm -rf test script src  # Eliminamos el boilerplate de ejemplo
```

---

### 7. Escribir el contrato paso a paso

En este paso crearemos el contrato `HelloWorld` dentro de `./src/HelloWorld.sol`, desglosando cada parte en fragmentos de código para entender su propósito.

---

#### 7.1. Cabecera y licencia

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;
```

* **SPDX Identifier**: `UNLICENSED` indica que el código es de prueba y no está bajo una licencia open source.
* **pragma solidity**: fija la versión de compilador (>=0.8.0 <0.9.0), para aprovechar mejoras de seguridad y optimizaciones.

---

#### 7.2. Declaración del contrato y variable de estado

```solidity
contract HelloWorld {
    address public owner;
```

* **contract HelloWorld**: define un nuevo contrato en la blockchain con el nombre `HelloWorld`.
* **address public owner**: declara una variable de estado `owner` que almacena la dirección del creador y genera automáticamente un getter público.

---

#### 7.3. Constructor

```solidity
    constructor() {
        owner = msg.sender;
    }
```

* **constructor()**: se ejecuta una sola vez, al momento del despliegue.
* **msg.sender**: representa la dirección que invoca la transacción; aquí se asigna a `owner` para registrar quién creó el contrato.

---

#### 7.4. Función de saludo

```solidity
    function greet(string memory name) external pure returns (string memory) {
        return string(abi.encodePacked("Hello, ", name, "!"));
    }
}
```

* **function greet(...)**:

  * `external`: accesible desde fuera del contrato.
  * `pure`: no lee ni modifica estado.
  * `returns (string memory)`: devuelve una cadena en memoria.
* **abi.encodePacked**: concatena eficientemente literales y parámetros en un solo `bytes`, luego convertido a `string`.

---

#### 7.5. Archivo completo

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

contract HelloWorld {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function greet(string memory name) external pure returns (string memory) {
        return string(abi.encodePacked("Hello, ", name, "!"));
    }
}
```

Ya tienes tu contrato listo para compilar y desplegar.

---

### 8. Compilar el contrato

```bash
forge build
```

---

### 9. Probar el contrato paso a paso

En esta sección validaremos que nuestro contrato `HelloWorld` se comporte como esperamos, escribiendo pruebas unitarias con Foundry.

---

#### 9.1. Crear el archivo de prueba

Crea un nuevo archivo en `test/HelloWorld.t.sol`:

```bash
mkdir -p test
nano test/HelloWorld.t.sol
```

Dentro de este archivo comenzamos con la cabecera y las importaciones necesarias:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Test.sol";
import "../src/HelloWorld.sol";
```

* `forge-std/Test.sol` provee utilidades de testing.
* Importamos nuestro contrato `HelloWorld` para instanciarlo en las pruebas.

---

#### 9.2. Configurar el entorno en `setUp()`

La función `setUp()` se ejecuta antes de cada prueba, ideal para inicializar estados compartidos:

```solidity
contract HelloWorldTest is Test {
    HelloWorld public hello;

    function setUp() public {
        hello = new HelloWorld();
    }
```

* Creamos una instancia fresca de `HelloWorld` en cada test.
* Guardamos la referencia en `hello` para usarla luego.

---

#### 9.3. Escribir la prueba `testGreet()`

Ahora verificamos la salida de la función `greet`:

```solidity
    function testGreet() public {
        // Llamamos greet con "Alice"
        string memory salida = hello.greet("Alice");
        // AssertEq compara dos strings y falla si no coinciden
        assertEq(salida, "Hello, Alice!");
    }
}
```

* `hello.greet("Alice")` debe devolver exactamente `Hello, Alice!`.
* `assertEq` marca la prueba como pasada o fallida.

---

#### 9.4. Ejecutar los tests

Guarda el archivo y en tu terminal lanza:

```bash
forge test
```

Foundry compilará automáticamente el contrato de pruebas y mostrará un resumen:

* ✅ `testGreet` — pasa correctamente si la salida coincide.
* ❌ Detalle de fallos si la prueba no arroja el resultado esperado.

Con esto confirmas que tu contrato funciona y que la función `greet` cumple con su propósito.

---

### 10. Despliegue paso a paso

En este paso automatizaremos el lanzamiento de tu contrato a la red Alfajores usando un script de Foundry.

---

#### 10.1. Crear la carpeta de scripts

Si aún no existe, crea la carpeta `script` en tu proyecto:

```bash
mkdir -p script
```

Luego abre un nuevo archivo de despliegue:

```bash
nano script/Deploy.s.sol
```

---

#### 10.2. Escribir el script de despliegue

Copia el siguiente contenido en `script/Deploy.s.sol`:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;

import "forge-std/Script.sol";
import "../src/HelloWorld.sol";

contract DeployScript is Script {
    function run() external {
        // Inicia el broadcast: todo lo que esté entre start/stop se firma y envía
        vm.startBroadcast();
        // Despliega el contrato HelloWorld
        HelloWorld deployed = new HelloWorld();
        // Opcional: emite un log con la dirección desplegada
        console.log("Contrato desplegado en:", address(deployed));
        vm.stopBroadcast();
    }
}
```

* **vm.startBroadcast()** y **vm.stopBroadcast()**: delimitan la sección que envía transacciones reales.
* **HelloWorld deployed = new HelloWorld()**: instancia y despliega el contrato.
* **console.log**: (requiere `import "forge-std/Script.sol";`) muestra la dirección en la terminal.

---

#### 10.3. Ejecutar el script

Usa `forge script` con tus variables de entorno:

```bash
forge script script/Deploy.s.sol \
  --rpc-url $CELO_NODE_URL \
  --broadcast \
  --account wallet-hello-celo
```

Explicación de flags:

* `--rpc-url`: URL de tu nodo (Alfajores Forno).
* `--broadcast`: activa la transmisión de transacciones a la red testnet.
* `--account`: alias o nombre del wallet importado en Foundry.

Al correr este comando, Foundry compilará el script, firmará la transacción con tu clave y enviará el despliegue. Verás en la salida la dirección del contrato.

---

#### 10.4. Guardar la dirección desplegada

Una vez tengas la dirección (ej. `0xAbC123...`), añade esta línea al final de tu `.env`:

```bash
echo "CELO_CONTRACT_ADDRESS=0xAbC123..." >> .env
```

Y recarga las variables:

```bash
source .env
```

Ahora `CELO_CONTRACT_ADDRESS` está lista para usar en las llamadas posteriores.

---

### 11. Interactuar con el contrato

```bash
source .env

cast call --rpc-url $CELO_NODE_URL $CELO_CONTRACT_ADDRESS \
  "greet(string)(string)" "Celo 😎"
```

Deberías ver:

```
"Hello, Celo 😎!"
```

---

## ¿Qué sigue después del despliegue?

Tu contrato ya está en la blockchain: **inmutable** y **público**. Lo puedes ver en CeloScan buscando la dirección desplegada para revisar transacciones, código verificado y funciones públicas.

---

## 📚 Recursos adicionales

* [Documentación de Celo](https://docs.celo.org/)
* [Docs de Celo CLI](https://docs.celo.org/cli)
* [Foundry Book](https://book.getfoundry.sh/)
* [Faucet Alfajores](https://alfajores.celo.org/faucet)
* [CeloScan Explorer](https://celoscan.io/)
* [Celo Composer – plantilla full‑stack](https://github.com/celo-org/celo-composer)
* [Awesome Celo – índice de recursos](https://github.com/jvgrootveld/awesome-celo)

---

## 🤝 Contribuciones

Esta guía hace parte de la iniciativa Celo Colombia Bootcamp. ¡Siéntete libre de forkear, modificar y usarla en tus proyectos o recursos de aprendizaje!

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Consulta el archivo [LICENSE](LICENSE) para más detalles.
