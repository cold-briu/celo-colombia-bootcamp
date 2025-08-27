# 🧪 Plantilla de Bootcamp - Dashboard de Uniswap

Un tutorial amigable para principiantes para construir y desplegar un dashboard de Uniswap v3 usando The Graph subgraph y React TypeScript. Esta plantilla de bootcamp proporciona una guía completa para desarrolladores Web3 que aprenden a interactuar con protocolos DeFi.

---

## 📘 Tabla de Contenidos

1. [📋 Prerrequisitos](#prerequisites)
   - 1.1. [🏗️ Crear Cuenta de Graph Studio](#context)
   - 1.2. [🔗 Conectar Wallet](#context)
   - 1.3. [📧 Agregar y Verificar Email](#context)
   - 1.4. [🔗 Obtener URL de Query del Subgraph de Uniswap V3](#context)

2. [🚀 Inicialización del Proyecto](#project-initialization)
   - 2.1. [⚛️ Crear Aplicación Next.js](#context)
   - 2.2. [🧹 Limpiar Plantilla por Defecto](#context)
   - 2.3. [✅ Verificar Configuración](#context)

3. [📊 Dashboard de Pool](#pool-dashboard)
   - 3.1. [🛤️ Crear Ruta API de Subgraph](#31-create-subgraph-api-route)
   - 3.2. [🧩 Crear Componente PoolDashboard](#32-create-pooldashboard-component)
   - 3.3. [🔧 Crear Servicio de Datos de Pool](#33-create-pool-data-service)
   - 3.4. [⚙️ Implementar Función getPoolData](#34-implement-getpooldata-function)
   - 3.5. [🎨 Implementación de UI](#35-ui-implementation)
     - 3.5.1. [❌ UI de Error](#351-error-ui)
     - 3.5.2. [⏳ Cargador de UI](#352-ui-loader)
     - 3.5.3. [📭 UI Sin Datos](#353-no-data-ui)
     - 3.5.4. [🖼️ Renderizar UI](#354-render-ui)

---

## 1. 📋 Prerrequisitos

#### Contexto
Antes de comenzar este tutorial, asegúrate de tener las herramientas y cuentas necesarias configuradas para construir un dashboard de Uniswap con integración de subgraph.

1.1. 🏗️ Crear una cuenta en [The Graph Studio](https://thegraph.com/studio/) y obtener una API key
   - Navegar al sitio web de The Graph Studio
   - Hacer clic en "Sign Up" para crear una nueva cuenta
   - Completar el proceso de registro

1.2. 🔗 Conectar wallet
   - Hacer clic en "Connect Wallet" en el dashboard de The Graph Studio
   - Seleccionar tu wallet preferido (MetaMask, WalletConnect, etc.)
   - Aprobar la solicitud de conexión en tu wallet

1.3. 📧 Agregar y verificar email
   - Ir a la configuración de tu cuenta en The Graph Studio
   - Agregar tu dirección de email para recibir notificaciones importantes
   - Revisar tu bandeja de entrada y hacer clic en el enlace de verificación
   - Confirmar que tu email está verificado en el dashboard

> 💡 **Nota**: La verificación de email es requerida para acceder a ciertas funciones y recibir notificaciones de uso de la API.


1.4 🔗 Obtener la URL de query del subgraph de Uniswap V3
   - Navegar a [The Graph Explorer - Uniswap V3 Subgraph](https://thegraph.com/explorer/subgraphs/ESdrTJ3twMwWVoQ1hUE2u7PugEHX3QkenudD6aXCkDQ4)
   - Localizar la sección "Query URL" en la página del subgraph
   - Copiar el endpoint de query: `/subgraphs/id/ESdrTJ3twMwWVoQ1hUE2u7PugEHX3QkenudD6aXCkDQ4`
   - La URL de query completa será: `https://gateway.thegraph.com/api/[api-key]/subgraphs/id/ESdrTJ3twMwWVoQ1hUE2u7PugEHX3QkenudD6aXCkDQ4`
       - Reemplazar `[api-key]` con tu API key obtenida del paso 1.1

> 💡 **Nota**: Este subgraph de Uniswap V3 (v0.0.3) proporciona acceso a datos de mainnet incluyendo pools, posiciones, swaps, e información de liquidez.


---

## 2. 🚀 Inicialización del Proyecto

#### Contexto
Configurar tu proyecto Next.js con TypeScript y Tailwind CSS, luego limpiar la plantilla por defecto para preparar la construcción de tu dashboard de Uniswap.

2.1. ⚛️ Crear una nueva aplicación Next.js con TypeScript y Tailwind CSS
   ```bash
   npx create-next-app@latest uniswap-dashboard --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
   ```
   - Navegar al directorio de tu proyecto:
   ```bash
   cd uniswap-dashboard
   ```

2.2. 🧹 Limpiar los archivos de la plantilla por defecto
   - Remover contenido innecesario de `src/app/page.tsx`:
   ```bash
   # Reemplazar todo el contenido con un punto de partida limpio
   ```
   - Eliminar el favicon por defecto y reemplazarlo con el tuyo propio (opcional)
   - Remover CSS no utilizado de `src/app/globals.css` (mantener imports de Tailwind)
   ```

2.3. ✅ Verificar tu configuración
   ```bash
   npm run dev
   ```
   - Abrir [http://localhost:3000](http://localhost:3000) en tu navegador
   - Confirmar que la aplicación se ejecuta sin errores
   - Deberías ver una página limpia lista para desarrollo

> ⚠️ **Nota**: Asegúrate de tener Node.js 18.17 o posterior instalado antes de ejecutar estos comandos.


---

## 3. 📊 Dashboard de Pool

#### Contexto
Crear un componente de dashboard que muestre datos específicos del pool usando el subgraph para mostrar información detallada del pool.

### 3.1. 🛤️ Crear Ruta API de Subgraph

Este componente requiere una llamada de subgraph, se requiere una ruta api para proteger las keys.

**3.1.1. 📁 Crear archivo de ruta**

Crear el archivo de ruta API para manejar las solicitudes de subgraph del lado del servidor.

```bash
code ./src/app/api/subgraph/route.ts
```

**3.1.2. 📥 Importar responses**

Importar los módulos necesarios de Next.js para manejar solicitudes API.

```typescript
import { NextRequest, NextResponse } from 'next/server';
```

**3.1.3. 📝 Crear export de ruta vacía**

Crear la estructura básica del manejador de ruta POST.

```typescript
export async function POST(req: NextRequest) {
  return new NextResponse();
}
```

**3.1.4. 🎯 Obtener query de la solicitud**

Extraer la query de GraphQL del cuerpo de la solicitud.

```typescript
const { query } = await req.json();
```

**3.1.5. 🌐 Hacer solicitud al subgraph**

Configurar la URL del subgraph y hacer la solicitud a The Graph.

```typescript
const SUBGRAPH_URL = 'https://gateway.thegraph.com/api/subgraphs/id/8cLf29KxAedWLVaEqjV8qKomdwwXQxjptBZFrqWNH5u2';

const response = await fetch(SUBGRAPH_URL, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${process.env.GRAPH_API_KEY!}`,
  },
  body: JSON.stringify({ query }),
});
```

**3.1.6. 🔄 Parsear response como JSON y retornar**

Parsear la respuesta del subgraph y retornarla al cliente.

```typescript
const data = await response.json();
return NextResponse.json(data);
```

**3.1.7. ⚠️ Capturar errores**

Envolver toda la función en un bloque try-catch para manejar cualquier error.

```typescript
try {
  // ... previous code ...
} catch (error) {
  console.error('Error fetching subgraph data:', error);
  return NextResponse.json(
    { error: 'Failed to fetch subgraph data' },
    { status: 500 }
  );
}
```

**3.1.8. 🔐 Crear archivo de variables de entorno**

Configurar el archivo de variables de entorno para almacenar de forma segura tu Graph API key.

```bash
# Crear archivo .env.local en la raíz de tu proyecto
touch .env.local
```

Agregar tu Graph API key a `.env.local`:

```bash
# .env.local
GRAPH_API_KEY=your-api-key-here
```

> ⚠️ **Nota de Seguridad**: Nunca hagas commit de tu API key real al control de versiones. El archivo `.env.local` es automáticamente ignorado por Next.js y debería ser agregado a tu archivo `.gitignore` si no está ya presente.

### 3.2. 🧩 Crear Componente PoolDashboard

**3.2.1. 📁 Crear nuevo archivo de componente PoolDashboard**

Crear un nuevo archivo de componente React para mostrar datos del pool.

```bash
code ./src/components/PoolDashboard.tsx
```

**3.2.2. 📝 Componente vacío que renderiza el nombre del componente como h1**

Crear una estructura básica de componente con un encabezado. Agregar la directiva "use client" ya que este componente usará React hooks.

```typescript
'use client';

export default function PoolDashboard() {
  return (
    <div>
      <h1>PoolDashboard</h1>
    </div>
  );
}
```

**3.2.3. 📥 Importar en la página**

Importar y usar el componente PoolDashboard en tu página principal.

```typescript
import PoolDashboard from '@/components/PoolDashboard';

export default function Home() {
  return (
    <main>
      <PoolDashboard />
    </main>
  );
}
```

**3.2.4. 🔄 Actualizar componente PoolDashboard para agregar manejadores de estado**

Importar React hooks y agregar manejo de estado para datos, error, y estados de carga.

```typescript
'use client';

import { useEffect, useState } from 'react';

export default function PoolDashboard() {
  const [data, setData] = useState<any>(null);
  const [err, setErr] = useState<string | null>(null);
  const [isLoading, setIsLoading] = useState(false);

  return (
    <div>
      <h1>PoolDashboard</h1>
    </div>
  );
}
```

**3.2.5. 📜 Crear constante de query**

Definir la query de GraphQL para obtener datos del pool desde el subgraph.

```typescript
const QUERY = `
  {
    liquidityPool(id: "0x357596DD7a0EF5CB703C5AAe4dA01EDFf176aE95") {
      name
      inputTokenBalances
      symbol
      totalValueLockedUSD
      id
      cumulativeSwapCount
    }
  }
`;
```

**3.2.6. 🚀 Crear función de obtención de datos**

Crear una función fetchData de marcador de posición que será implementada más tarde usando la capa de servicio.

```typescript
const fetchData = async () => {
  // Esto será implementado en la sección 3.4 usando la capa de servicio
};
```

**3.2.6.1. ⏰ Agregar useEffect para disparar la obtención de datos**

Usar useEffect para obtener automáticamente los datos cuando el componente se monta.

```typescript
useEffect(() => {
  fetchData();
}, []);
```

**3.2.7. 🖼️ Agregar lógica de renderizado**

**3.2.7.1. ❌ Manejar estado de error**

Mostrar mensaje de error si hay un error obteniendo datos.

```typescript
if (err) {
  return (
    <div>
      <h1>PoolDashboard</h1>
      <p>Error: {err}</p>
    </div>
  );
}
```

**3.2.7.2. ⏳ Manejar estado de carga**

Mostrar mensaje de carga mientras los datos están siendo obtenidos.

```typescript
if (isLoading) {
  return (
    <div>
      <h1>PoolDashboard</h1>
      <p>Loading...</p>
    </div>
  );
}
```

**3.2.7.3. 📊 Manejar visualización de datos**

Mostrar los datos obtenidos como JSON formateado cuando estén disponibles.

```typescript
return (
  <div>
    <h1>PoolDashboard</h1>
    {data && (
      <pre>
        <span>{JSON.stringify(data, null, 2)}</span>
      </pre>
    )}
  </div>
);
```

### 3.3. 🔧 Crear Servicio de Datos de Pool

**3.3.1. ➕ Agregar función getPoolData al servicio de subgraph**

Agregar una nueva función para manejar solicitudes de datos de pool en el archivo de servicio de subgraph existente.

```bash
code ./src/lib/subgraph-service.ts
```

**3.3.2. 🔨 Crear función getPoolData**

**3.3.2.1. 📝 Crear función async await vacía**

Crear la estructura básica de la función con patrón async/await y definir la query del pool dentro del servicio.

```typescript
const POOL_QUERY = `
  {
    liquidityPool(id: "0x357596DD7a0EF5CB703C5AAe4dA01EDFf176aE95") {
      name
      inputTokenBalances
      symbol
      id
      cumulativeSwapCount
    }
  }
`;

export async function getPoolData() {
  // La implementación de la función irá aquí
}
```

**3.3.2.2. 🌐 Agregar fetch y parsear datos**

Implementar la solicitud fetch y lógica de parseo JSON usando la query del pool definida.

```typescript
export async function getPoolData() {
  const res = await fetch('/api/subgraph', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ query: POOL_QUERY }),
  });
  if (!res.ok) throw new Error(`${res.status} ${res.statusText}`);
  const json = await res.json();
  return json.data;
}
```

**3.3.2.3. ⚠️ Manejar error**

Envolver la función en un bloque try-catch para manejar cualquier error que ocurra.

```typescript
export async function getPoolData() {
  try {
    const res = await fetch('/api/subgraph', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ query: POOL_QUERY }),
    });
    if (!res.ok) throw new Error(`${res.status} ${res.statusText}`);
    const json = await res.json();
    return json.data;
  } catch (e: any) {
    throw new Error(e.message);
  }
}
```

### 3.4. ⚙️ Implementar Función getPoolData

**3.4.1. 📥 Importar al componente**

Importar la función de servicio getPoolData al componente PoolDashboard.

```typescript
'use client';

import { useEffect, useState } from 'react';
import { getPoolData } from '@/lib/subgraph-service';
```

**3.4.2. 📝 Crear función fetchData vacía**

Crear una función de marcador de posición para manejar la obtención de datos dentro del componente.

```typescript
const fetchData = async () => {
  // La lógica de obtención de datos irá aquí
};
```

**3.4.3. ⚙️ Try catch y finally getPoolData y actualizar loaders**

Implementar la función fetchData con estados de carga apropiados y manejo de errores usando el servicio getPoolData.

```typescript
const fetchData = async () => {
  setIsLoading(true);
  setErr(null);
  
  try {
    const result = await getPoolData();
    setData(result);
  } catch (e: any) {
    setErr(e.message);
  } finally {
    setIsLoading(false);
  }
};
```

**3.4.4. ⏰ Implementar fetchData en useEffect**

Llamar la función fetchData cuando el componente se monta usando useEffect.

```typescript
useEffect(() => {
  fetchData();
}, []);
```

### 3.5. 🎨 Implementación de UI

#### Contexto
Mejorar el componente PoolDashboard con estilos de UI apropiados y manejo de errores para crear un dashboard de apariencia profesional con estados de error amigables al usuario.

**3.5.1. ❌ UI de Error**

Implementar una UI de manejo de errores comprensiva que proporcione retroalimentación clara a los usuarios cuando falla la obtención de datos.

**3.5.1.1. 🎨 Estilizar el estado de error**

Reemplazar la visualización básica de error con una UI de error estilizada que incluya indicadores visuales y acciones del usuario.

```typescript
if (err) {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 border-l-4 border-red-500">
      <h2 className="text-xl font-semibold text-red-700 mb-2">Error Loading Pool Data</h2>
      <p className="text-red-600">{err}</p>
      <button
        onClick={fetchData}
        className="mt-4 px-4 py-2 bg-red-600 text-white rounded-md hover:bg-red-700 transition-colors"
      >
        Retry
      </button>
    </div>
  );
}
```

**3.5.1.2. 📭 Agregar estado sin datos**

Manejar casos donde la API retorna exitosamente pero sin datos de pool disponibles.

```typescript
if (!data || !data.liquidityPool) {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 border-l-4 border-yellow-500">
      <h2 className="text-xl font-semibold text-yellow-700 mb-2">No Pool Data Available</h2>
      <p className="text-yellow-600">Unable to load pool information at this time.</p>
      <button
        onClick={fetchData}
        className="mt-4 px-4 py-2 bg-yellow-600 text-white rounded-md hover:bg-yellow-700 transition-colors"
      >
        Retry
      </button>
    </div>
  );
}
```

> 💡 **Nota de Diseño**: La UI de error usa clases de Tailwind CSS para crear una apariencia profesional con:
> - Borde y colores rojos para errores críticos
> - Colores amarillos/ámbar para estados de advertencia (sin datos)
> - Mensajería clara y botones de reintentar accionables
> - Espaciado y tipografía consistentes

**3.5.2. ⏳ Cargador de UI**

Implementar un estado de carga elegante con marcadores de posición de esqueleto animados para proporcionar retroalimentación visual mientras los datos están siendo obtenidos.

**3.5.2.1. 🎨 Estilizar el estado de carga**

Reemplazar el mensaje básico de carga con un cargador de esqueleto animado que imite la estructura del contenido.

```typescript
if (isLoading) {
  return (
    <div className="bg-white rounded-lg shadow-md p-6">
      <div className="animate-pulse">
        <h2 className="text-xl font-semibold text-gray-700 mb-4">Loading Pool Data...</h2>
        <div className="space-y-3">
          <div className="h-4 bg-gray-200 rounded w-3/4"></div>
          <div className="h-4 bg-gray-200 rounded w-1/2"></div>
          <div className="h-4 bg-gray-200 rounded w-2/3"></div>
        </div>
      </div>
    </div>
  );
}
```

> 💡 **Nota de Diseño**: La UI de carga presenta:
> - Efecto de pulso animado usando la clase `animate-pulse` de Tailwind
> - Marcadores de posición de esqueleto con anchos variados para simular contenido
> - Estilo consistente con el layout principal del dashboard
> - Mensaje de carga claro para accesibilidad

**3.5.3. 📭 UI Sin Datos**

Implementar un estado sin datos amigable al usuario que maneje casos donde la API retorna exitosamente pero no contiene información de pool.

**3.5.3.1. 🎨 Estilizar el estado sin datos**

Crear un componente UI de estilo de advertencia que comunique claramente cuando los datos del pool no están disponibles y proporcione opciones de recuperación.

```typescript
if (!data || !data.liquidityPool) {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 border-l-4 border-yellow-500">
      <h2 className="text-xl font-semibold text-yellow-700 mb-2">No Pool Data Available</h2>
      <p className="text-yellow-600">Unable to load pool information at this time.</p>
      <button
        onClick={fetchData}
        className="mt-4 px-4 py-2 bg-yellow-600 text-white rounded-md hover:bg-yellow-700 transition-colors"
      >
        Retry
      </button>
    </div>
  );
}
```

> 💡 **Nota de Diseño**: La UI sin datos proporciona:
> - Esquema de colores amarillo/ámbar para indicar un estado de advertencia (no un error)
> - Mensajería clara explicando la situación a los usuarios
> - Funcionalidad de reintentar para intentar cargar datos nuevamente
> - Estilo consistente con otros estados de UI para una experiencia de usuario cohesiva

**3.5.4. 🖼️ Renderizar UI**

Implementar la UI principal del dashboard que muestre datos del pool en un layout profesional y organizado con jerarquía visual clara.

**3.5.4.1. 🏢 Crear la estructura principal del dashboard**

Reemplazar la visualización JSON con un layout de dashboard estructurado que incluya secciones de encabezado, métricas y contenido.

```typescript
const pool = data.liquidityPool;

return (
  <div className="bg-white rounded-lg shadow-md overflow-hidden">
    {/* Encabezado */}
    <div className="bg-blue-50 px-6 py-4 border-b">
      <h2 className="text-2xl font-bold text-blue-900">{pool.name || 'Uniswap Pool'}</h2>
      <p className="text-blue-700 text-sm mt-1">{pool.symbol || 'N/A'}</p>
      <p className="text-blue-600 text-xs font-mono mt-2">
        ID: {shortenAddress(pool.id)}
      </p>
    </div>

    {/* Contenido Principal */}
    <div className="p-6">
      {/* Cuadrícula de Métricas Clave */}
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6">
        <div className="bg-purple-50 p-4 rounded-lg border border-purple-200">
          <h3 className="text-sm font-medium text-purple-700 mb-1">Cumulative Swaps</h3>
          <p className="text-2xl font-bold text-purple-900">
            {formatNumber(pool.cumulativeSwapCount || 0)}
          </p>
        </div>

        <div className="bg-indigo-50 p-4 rounded-lg border border-indigo-200">
          <h3 className="text-sm font-medium text-indigo-700 mb-1">Pool ID</h3>
          <p className="text-sm font-mono text-indigo-900 break-all">
            {pool.id}
          </p>
        </div>
      </div>

      {/* Balances de Tokens */}
      <div className="mb-6">
        <h3 className="text-lg font-semibold text-gray-900 mb-3">Token Balances</h3>
        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          <div className="bg-gray-50 p-4 rounded-lg border">
            <h4 className="text-sm font-medium text-gray-700 mb-2">Token 0 Balance</h4>
            <p className="text-lg font-mono text-gray-900">
              {formatTokenBalance(pool.inputTokenBalances?.[0] || '0', 0)}
            </p>
          </div>

          <div className="bg-gray-50 p-4 rounded-lg border">
            <h4 className="text-sm font-medium text-gray-700 mb-2">Token 1 Balance</h4>
            <p className="text-lg font-mono text-gray-900">
              {formatTokenBalance(pool.inputTokenBalances?.[1] || '0', 1)}
            </p>
          </div>
        </div>
      </div>

      {/* Botón de Actualizar */}
      <div className="flex justify-end">
        <button
          onClick={fetchData}
          className="px-4 py-2 bg-blue-600 text-white rounded-md hover:bg-blue-700 transition-colors focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
          disabled={isLoading}
        >
          {isLoading ? 'Refreshing...' : 'Refresh Data'}
        </button>
      </div>
    </div>
  </div>
);
```

> 💡 **Nota de Diseño**: La UI principal del dashboard presenta:
> - Sección de encabezado limpia con nombre del pool, símbolo e ID acortado
> - Tarjetas de métricas codificadas por color para diferentes tipos de datos
> - Layout de cuadrícula responsivo que se adapta a diferentes tamaños de pantalla
> - Estilo profesional with espaciado y tipografía consistentes
> - Botón de actualizar interactivo con estados de carga y características de accesibilidad