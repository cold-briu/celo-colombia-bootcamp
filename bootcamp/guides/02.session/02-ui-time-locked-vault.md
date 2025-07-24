# Guía de configuración UI para Time-Lock Vault

Una guía paso a paso para construir una UI web3 mínima para conexión de wallet usando Next.js, TypeScript, Tailwind CSS y Viem.

## 📑 Índice

1. [Prerrequisitos](#prerrequisitos)
2. [Estructuración del proyecto](#1-estructuración-del-proyecto)
3. [Limpieza de plantilla](#2-limpieza-de-plantilla)
4. [Importación de ABI](#3-importación-de-abi)
5. [Instalación de dependencias](#4-instalación-de-dependencias)
6. [Configuración del cliente Viem](#6-configuración-del-cliente-viem)
7. [Página de prueba Hello World](#7-página-de-prueba-hello-world)
8. [Configuración de conexión de wallet](#8-configuración-de-conexión-de-wallet)
   - 8.1 [Crear el archivo del componente principal](#81-crear-el-archivo-del-componente-principal)
   - 8.2 [Agregar componente de estado de carga](#82-agregar-componente-de-estado-de-carga)
   - 8.3 [Agregar diseño de contenedor principal y encabezado](#83-agregar-diseño-de-contenedor-principal-y-encabezado)
   - 8.4 [Agregar botón de conexión de wallet](#84-agregar-botón-de-conexión-de-wallet)
   - 8.5 [Agregar indicador de estado de conexión](#85-agregar-indicador-de-estado-de-conexión)
   - 8.6 [Agregar lógica de conexión de wallet](#86-agregar-lógica-de-conexión-de-wallet)
   - 8.7 [Agregar visualización de saldo de cuenta](#87-agregar-visualización-de-saldo-de-cuenta)
9. [Creación de bóveda](#9-creación-de-bóveda-como-componente)
   - 9.1 [Crear archivo del componente VaultCreation](#91-crear-archivo-del-componente-vaultcreation)
   - 9.2 [Importar y usar el componente VaultCreation en la página principal](#92-importar-y-usar-el-componente-vaultcreation-en-la-página-principal)
   - 9.3 [Eliminar lógica de creación de bóveda de la página principal](#93-eliminar-lógica-de-creación-de-bóveda-de-la-página-principal)
10. [Listado de bóvedas](#10-listado-de-bóvedas)
    - 10.1 [Crear archivo del componente VaultList](#101-crear-archivo-del-componente-vaultlist)
    - 10.2 [Importar y usar el componente VaultList](#102-importar-y-usar-el-componente-vaultlist)
    - 10.3 [Actualizar VaultCreation para refrescar la lista](#103-actualizar-vaultcreation-para-refrescar-la-lista)
11. [Retirar bóveda](#11-retirar-bóveda)
    - 11.1 [Agregar estado y función de retiro](#111-agregar-estado-y-función-de-retiro)
    - 11.2 [Agregar lógica del botón de retiro](#112-agregar-lógica-del-botón-de-retiro)
    - 11.3 [Actualizar página principal para pasar cliente de wallet](#113-actualizar-página-principal-para-pasar-cliente-de-wallet)
    - 11.4 [Probar el flujo de retiro](#114-probar-el-flujo-de-retiro)
12. [Compilar y ejecutar](#12-compilar-y-ejecutar)

---

## Prerrequisitos

* Node.js y npm
* Dirección desplegada del contrato `TimeLockVaultFactory`
* Cuenta de testnet Alfajores de Celo
* Billetera de navegador (MetaMask, Valora, etc.)

---

## 1. Estructuración del proyecto

Crea un nuevo proyecto Next.js con TypeScript, Tailwind CSS y ESLint:

```bash
npx create-next-app@latest ui-time-lock-vault --ts --tailwind --eslint --yes
cd ui-time-lock-vault
```

---

## 2. Limpieza de plantilla

Elimina archivos predeterminados para empezar limpio, pero mantén `globals.css`:

```bash
rm -rf src/app/favicon.ico src/app/page.tsx
```

---

## 3. Importación de ABI

Crea el directorio ABI y copia el ABI del contrato desde tu build de Foundry:

```bash
mkdir abi
cp ../time-lock-vault/out/TimeLockVaultFactory.sol/TimeLockVaultFactory.json abi/
```

---

## 4. Instalación de dependencias

Instala Viem para interacciones blockchain y tipos adicionales de TypeScript:

```bash
npm install viem
npm install -D @types/node
```

---

## 6. Configuración del cliente Viem

Crea la configuración del cliente Viem para Celo Alfajores:

### Crear `src/lib/viem.ts`

```ts
declare global {
  interface Window {
    ethereum?: unknown;
  }
}

import { createPublicClient, createWalletClient, custom } from 'viem'
import { celoAlfajores } from 'viem/chains'

function getTransport() {
  if (typeof window === 'undefined') {
    throw new Error('No window.ethereum: debe ejecutarse en navegador');
  }
  if (!window.ethereum) {
    throw new Error('No wallet inyectada encontrada');
  }
  // eslint-disable-next-line @typescript-eslint/no-explicit-any
  return custom(window.ethereum as any);
}

export const publicClient = createPublicClient({
  chain: celoAlfajores,
  transport: getTransport(),
})

export const walletClient = createWalletClient({
  chain: celoAlfajores,
  transport: getTransport(),
})
```

---

## 7. Página de prueba Hello World

Antes de implementar la UI web3 completa, vamos a crear una página simple "Hello World" para verificar que la configuración básica de Next.js funciona correctamente.

### Crear `src/app/page.tsx`

Crea una página simple para probar la configuración básica:

```tsx
export default function Home() {
  return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 flex items-center justify-center">
      <div className="max-w-md w-full mx-auto p-6">
        <div className="bg-white rounded-xl shadow-lg p-6 text-center">
          <h1 className="text-3xl font-bold text-gray-900 mb-4">¡Hola Mundo!</h1>
          <p className="text-gray-600 mb-6">Tu aplicación Next.js está funcionando correctamente.</p>
          <div className="bg-green-50 border border-green-200 rounded-lg p-4">
            <p className="text-sm text-green-800 font-medium">✅ Configuración completa</p>
            <p className="text-xs text-green-600">Listo para construir funciones web3</p>
          </div>
        </div>
      </div>
    </div>
  )
}
```

### Probar la configuración

Ejecuta el servidor de desarrollo para verificar que todo funciona:

```bash
npm run dev
```

Visita `http://localhost:3000` en tu navegador. Deberías ver:
- Un mensaje "¡Hola Mundo!" centrado
- El mismo fondo degradado y estilo de tarjeta que se usará en la UI web3
- Un indicador verde de éxito mostrando que la configuración está completa

Este paso asegura que:
- Next.js está configurado correctamente
- Tailwind CSS está funcionando
- El enfoque básico de estilos es funcional
- El servidor de desarrollo puede iniciar sin errores

Una vez que confirmes que esta página se renderiza correctamente, puedes proceder a la implementación de UI mínima.

## 8. Configuración de conexión de wallet

Crea el componente UI principal con conexión de wallet siguiendo estos sub-pasos:

### 8.1 Crear el archivo del componente principal

Crea `src/pages/index.tsx` con la estructura básica del componente:

```tsx
"use client";
import { useState, useRef, useEffect } from 'react'
import abiJson from '../../abi/TimeLockVaultFactory.json'
import type { Abi } from 'viem'

const abi = abiJson.abi as Abi

export default function Home() {
    const [isClient, setIsClient] = useState(false)

    // Refs para mantener clientes viem, solo se establecen en cliente
    const walletClient = useRef<any>(null)

    useEffect(() => {
        setIsClient(true)
        // Importación dinámica para evitar SSR
        import('../lib/viem').then(({ walletClient: wc }) => {
            walletClient.current = wc
        })
    }, [])

    // El componente se construirá en sub-pasos a continuación
    return <div>Cargando...</div>
}
```

### 8.2 Agregar componente de estado de carga

Agrega el esqueleto de carga que se muestra durante la hidratación del lado del cliente:

```tsx
// Mostrar estado de carga hasta que se complete la hidratación del lado del cliente
if (!isClient) {
    return (
        <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 flex items-center justify-center">
            <div className="max-w-md w-full mx-auto p-6">
                <div className="bg-white rounded-xl shadow-lg p-6 space-y-4">
                    <div className="animate-pulse">
                        <div className="h-12 bg-gray-200 rounded-lg mb-4"></div>
                        <div className="text-center text-gray-500">Cargando...</div>
                    </div>
                </div>
            </div>
        </div>
    )
}
```

### 8.3 Agregar diseño de contenedor principal y encabezado

Agrega el contenedor principal con fondo degradado, tarjeta centrada y sección de encabezado:

```tsx
return (
    <div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 flex items-center justify-center p-4">
        <div className="max-w-md w-full mx-auto">
            <div className="bg-white rounded-xl shadow-lg p-6 space-y-6">
                {/* Encabezado */}
                <div className="text-center">
                    <h1 className="text-2xl font-bold text-gray-900 mb-2">Wallet Web3</h1>
                    <p className="text-gray-600">Conecta tu wallet para comenzar</p>
                </div>
                
                {/* El contenido se agregará en los siguientes sub-pasos */}
            </div>
        </div>
    </div>
)
```

### 8.4 Agregar botón de conexión de wallet

Agrega el botón de conexión con icono, estado deshabilitado cuando está conectado y etiqueta dinámica:

```tsx
{/* Conexión de Wallet */}
<div className="space-y-3">
    <button 
        onClick={onConnect}
        disabled={!!account}
        className="w-full px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 transition-colors duration-200 font-medium flex items-center justify-center space-x-2 disabled:bg-gray-400 disabled:cursor-not-allowed"
    >
        <svg className="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
        </svg>
        <span>{account ? 'Wallet Conectada' : 'Conectar Wallet'}</span>
    </button>
</div>
```

### 8.5 Agregar indicador de estado de conexión

Agrega el estado de conexión que muestra tanto estados conectados como no conectados:

```tsx
{/* Estado de conexión */}
{account ? (
    <div className="bg-green-50 border border-green-200 rounded-lg p-3">
        <p className="text-sm text-green-800 font-medium">Conectada</p>
        <p className="text-xs text-green-600 font-mono break-all">{account}</p>
    </div>
) : (
    <div className="bg-gray-50 border border-gray-200 rounded-lg p-3">
        <p className="text-sm text-gray-800 font-medium">No conectada</p>
        <p className="text-xs text-gray-600">Haz clic en el botón de arriba para conectar tu wallet</p>
    </div>
)}
```

### 8.6 Agregar lógica de conexión de wallet

Agrega el estado de cuenta y la funcionalidad de conexión de wallet:

```tsx
// Agregar estado de cuenta
const [account, setAccount] = useState<string>()

// Función de conexión de wallet
async function onConnect() {
    if (!walletClient.current) return
    const addresses = await walletClient.current.requestAddresses()
    setAccount(addresses[0])
}
```

### 8.7 Agregar visualización de saldo de cuenta

Agrega la visualización del saldo que muestra el saldo de CELO del usuario cuando está conectado:

```tsx
// Agregar estado y función de obtención de saldo
const [balance, setBalance] = useState<string>('0')

// Función de obtención de saldo
async function fetchBalance(address: string) {
    if (!publicClient.current) return
    const balanceWei = await publicClient.current.getBalance({ address })
    const balanceCelo = Number(balanceWei) / 1e18
    setBalance(balanceCelo.toFixed(4))
}

// Actualizar función de conexión para obtener saldo
async function onConnect() {
    if (!walletClient.current) return
    const addresses = await walletClient.current.requestAddresses()
    setAccount(addresses[0])
    await fetchBalance(addresses[0])
}

// Agregar visualización de saldo después del estado de conexión
{/* Visualización de saldo */}
{account && (
    <div className="bg-blue-50 border border-blue-200 rounded-lg p-3">
        <p className="text-sm text-blue-800 font-medium">Saldo de cuenta</p>
        <p className="text-lg text-blue-900 font-bold">{balance} CELO</p>
    </div>
)}
```

---

## 9. Creación de bóveda (como componente)

Ahora que la conexión de wallet funciona, agreguemos la funcionalidad de creación de bóveda como un componente separado. Esto permitirá a los usuarios crear bóvedas con bloqueo temporal con CELO y mantener tu página principal limpia y modular.

### 9.1 Crear archivo del componente VaultCreation

Crea un nuevo archivo en `src/app/VaultCreation.tsx`:

[Incluye aquí el código completo del componente VaultCreation que ya tienes]

### 9.2 Importar y usar el componente VaultCreation en la página principal

En tu página principal, importa y usa el nuevo componente:

```tsx
import VaultCreation from "./VaultCreation";
// ... dentro del return de tu componente Home, después de la conexión de wallet ...
<VaultCreation 
  account={account} 
  walletClient={walletClient.current} 
  publicClient={publicClient.current} 
  balance={balance} 
/>
```

### 9.3 Eliminar lógica de creación de bóveda de la página principal

Elimina el estado, lógica y UI de creación de bóveda de tu página principal, ya que ahora vive en el componente `VaultCreation`.

---

## 10. Listado de bóvedas

Ahora creemos un componente para mostrar las bóvedas de bloqueo temporal existentes. Esto permitirá a los usuarios ver todas las bóvedas que se han creado, incluyendo las suyas propias y las de otros.

### 10.1 Crear archivo del componente VaultList

[Incluye aquí el código completo del componente VaultList que ya tienes]

### 10.2 Importar y usar el componente VaultList

En tu página principal, importa y agrega el nuevo componente:

```tsx
import VaultList from "./VaultList";
// ... dentro del return de tu componente Home, después de VaultCreation ...
<VaultList 
  account={account} 
  publicClient={publicClient.current} 
  walletClient={walletClient.current}
/>
```

### 10.3 Actualizar VaultCreation para refrescar la lista

En tu componente `VaultCreation.tsx`, agrega un prop de callback para refrescar la lista de bóvedas después de la creación exitosa.

---

## 11. Retirar bóveda

Agrega funcionalidad de retiro al componente VaultList para completar el ciclo de vida de la bóveda con bloqueo temporal.

### 11.1 Agregar estado y función de retiro

En `src/app/VaultList.tsx`, agrega estado y función de retiro:

[Incluye aquí el código de estado y función de retiro]

### 11.2 Agregar lógica del botón de retiro

Agrega botón de retiro a cada tarjeta de bóveda que cumpla los criterios:

[Incluye aquí el código del botón de retiro]

### 11.3 Actualizar página principal para pasar cliente de wallet

En tu página principal, pasa el cliente de wallet a VaultList:

```tsx
<VaultList 
  account={account} 
  publicClient={publicClient.current} 
  walletClient={walletClient.current}
/>
```

### 11.4 Probar el flujo de retiro

1. Crea una bóveda con duración corta (ej. 30 segundos)
2. Espera a que pase el tiempo de desbloqueo
3. Haz clic en el botón **Retirar**
4. Confirma la transacción en tu wallet
5. Verifica que el estado de la bóveda se actualice a "Retirada"

Esto completa la funcionalidad básica de la bóveda con bloqueo temporal: crear, listar y retirar.

---

## 12. Compilar y ejecutar

### Actualizar configuración de TypeScript

Asegúrate de que tu `tsconfig.json` apunte a ES2020 o superior para soporte de BigInt:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    // ... otras opciones
  }
}
```

### Compilar la aplicación

```bash
npm run build
```

### Iniciar servidor de desarrollo

```bash
npm run dev
```

---

## Recursos adicionales

- [Documentación de Next.js](https://nextjs.org/docs)
- [Documentación de Viem](https://viem.sh/)
- [Documentación de Celo](https://docs.celo.org/)

**🎉 ¡Felicitaciones!** Has construido exitosamente una interfaz de usuario completa para interactuar con tu contrato inteligente de bóveda con bloqueo temporal en Celo Alfajores.
