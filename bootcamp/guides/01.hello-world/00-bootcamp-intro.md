# 📝 Introduccion

## 🚀 Visión general del bootcamp

El **Celo DeFi Builder Bootcamp** tiene como objetivo transformar desarrolladores **Web2** junior de Colombia en **builders** del ecosistema **Celo**, fomentando la colaboración en la comunidad **Celo Colombia** y preparando a los participantes para oportunidades en grants, empleo y liderazgo técnico.


---

## 🔎 ¿Qué es blockchain, Ethereum, contratos inteligentes y Celo?

Para entender Web3 desde tu experiencia en Web2, vamos a comparar conceptos conocidos con esta nueva arquitectura.

---

### 1. Blockchain vs Servidor Centralizado

* **Registro Inmutable**: Imagina una base de datos en la nube donde cada registro se firma criptográficamente y queda sellado para siempre. Una vez escrito, nadie puede alterarlo.
* **Infraestructura Descentralizada**: En lugar de depender de un único servidor (p. ej., tu API REST en AWS), decenas o cientos de **nodos** mantienen copias sincronizadas de esos datos.
* **Coordinación sin Conocimiento Mutuo**: Los nodos se conectan con un protocolo P2P, comparten transacciones en un **mempool** y usan un mecanismo de **consenso** (por ejemplo, Proof of Stake) para decidir qué bloque de transacciones se agrega a la cadena. No necesitan saber nada de quién maneja cada nodo.
* **Transparencia y Resistencia a la Censura**: Cada bloque es revisado por la mayoría de los nodos antes de añadirse, evitando modificaciones maliciosas.

*Aprende más: [Blockchain – Wikipedia](https://en.wikipedia.org/wiki/Blockchain)*

---

### 2. Ethereum vs AWS

* **Más que una criptomoneda**: Bitcoin es útil para transferir valor (como PayPal), pero Ethereum es una plataforma completa de **smart contracts**, capaz de ejecutar aplicaciones descentralizadas sin servidores tradicionales.
* **Analogía Web2**: Imagina AWS como un catálogo de servicios — EC2 para servidores, S3 para almacenamiento, RDS para bases de datos — donde gestionas instancias y escalas según demanda. En Ethereum, despliegas smart contracts directamente en la red global, pagas solo por el cómputo consumido (gas) y no te preocupas por mantenimiento de infraestructura ni escalabilidad.
* **Ecosistema DeFi**: Protocolos como Uniswap y Aave funcionan como servicios financieros en AWS Marketplace (por ejemplo, un servicio de exchange o lending), pero sin intermediarios: las operaciones se ejecutan automáticamente mediante smart contracts, asegurando transparencia y acceso global.

*Aprende más: [What is Ethereum? – Ethereum.org](https://ethereum.org/en/what-is-ethereum/)*

---

### 3. Contratos inteligentes como Microservicios

* **Definición**: Código auto-ejecutable que vive en la blockchain y se activa cuando se cumplen condiciones.
* **Paralelo con Microservicios**:

  * Cada contrato expone funciones (endpoints) y lógica de negocio, similar a un servicio independiente en tu arquitectura.
  * La comunicación entre contratos ocurre on-chain, parecido a llamar APIs REST o gRPC entre servicios.
  * Una vez desplegado, el contrato es inmutable y versionado (como un contenedor Docker inalterable).
* **Casos de uso**:

  * **DeFi**: Pools de liquidez, préstamos y swaps sin intermediarios.
  * **NFT**: Implementación de tokens únicos para arte digital, tickets, etc.

*Aprende más: [Smart contract – Wikipedia](https://en.wikipedia.org/wiki/Smart_contract)*

---

### 4. Celo como Layer 2 de Ethereum

* **¿Qué aporta?**: Rollups zk-integrados para escalar más de 2 000 TPS con gas casi nulo, manteniendo la seguridad de Ethereum.
* **Enfoque móvil**: Direcciones basadas en número de teléfono y experiencia optimizada para apps, ideal para usuarios sin experiencia crypto.
* **Ventajas frente a otras L2**:

  * EVM nativo, facilita portar dApps existentes.
  * Stablecoins colateralizadas on-chain: **CELO** (gobernanza y staking), **cUSD**, **cEUR** y **cCOP** (pagos sin volatilidad).
  * Gas pagado en cualquier de estos tokens, permitiendo abstraer la complejidad de la criptomoneda.
  * Lanzamiento v1.5.0 (1 jul 2025): mejoras en agregación de transacciones y pagos off-chain.

*Aprende más: [Celo Layer 2 Release – Documentación de Celo](https://docs.celo.org/what-is-celo#layer-2-release)* 


---

## 🎯 Propósito y misión

Este curso, impulsado desde la comunidad de **Celo Colombia**, busca formar a nuevos desarrolladores Web3 en **Celo**. Desde el primer día, enseña a acceder a grants y rutas de financiamiento de bienes públicos, incentivando la creación de soluciones que respondan a su propio contexto y construyan en **Celo**.

---


## 📚 Contenido de las sesiones


1. **Sesión 1: Hello Celo, Proof of Ship y configuración de Karma GAP**  
   * Introducción a **Proof of Ship**.  
   * Registro en **Karma GAP**.
   * Creación y despliegue del primer contrato **Hello World**.  

2. **Sesión 2: Time-Locked Vault con UI**  
   * Desarrollo de un contrato inteligente de vault con bloqueo temporal.  
   * Construcción de una interfaz de usuario para interactuar con el vault.

3. **Sesión 3: Contrato multisig y UI con Next.js**  
   * Creación de un contrato inteligente multisig para gestionar firmas múltiples.  
   * Desarrollo del frontend en **Next.js** para interactuar con el contrato y procesar transacciones en **cCOP**.

4. **Sesión 4: Dashboard de LP de Uniswap en Celo**  
   * Construcción de un dashboard que muestre métricas clave de un pool de liquidez de **Uniswap V3** en Celo, incluyendo APR y riesgos de impermanent loss.


---

## 🔐 Creación de tu primera wallet en Celo

1. **Instalación y configuración de MetaMask**  
   * Ve al sitio oficial de MetaMask y agrega la extensión a tu navegador.  
   * Crea una nueva wallet siguiendo las instrucciones en pantalla.  
2. **Conexión de MetaMask a la red Alfajores vía Chainlist**  
   * Ingresa a `https://chainlist.org`.  
   * Busca **Celo Alfajores**, haz clic en **Connect Wallet** y luego en **Add to MetaMask**.  
3. **Generación de tu primera cuenta y claves**  
   * En MetaMask, selecciona la red **Alfajores**.  
   * Copia tu **dirección pública** y guarda tu **frase secreta** en un lugar seguro.  
   > **Tip:** Nunca compartas tu frase secreta y respáldala en un medio offline.
