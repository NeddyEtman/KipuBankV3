# KipuBankV3
# KipuBank V3 - DeFi Aggregator Vault (Zap to USDC)

![License](https://img.shields.io/badge/License-MIT-blue.svg)
![Solidity](https://img.shields.io/badge/Solidity-%5E0.8.26-363636)
![Protocol](https://img.shields.io/badge/Integration-Uniswap_V2-ff007a)
![Status](https://img.shields.io/badge/Audit-Experimental-orange)

**KipuBankV3** transforma la bóveda tradicional en un **Agregador DeFi inteligente**. A diferencia de las versiones anteriores que custodiaban activos volátiles, la V3 acepta depósitos en cualquier token (ETH, WBTC, UNI, etc.), los intercambia automáticamente a **USDC** utilizando **Uniswap V2**, y acredita el valor estable en el balance del usuario.

---

## 🚀 Mejoras de Alto Nivel (V2 vs V3)

La evolución de V2 a V3 representa un cambio de paradigma: de "Custodia Multiactivo" a "Gestión de Liquidez Automatizada".

| Característica | KipuBank V2 (Bóveda) | KipuBank V3 (Agregador) | ¿Por qué el cambio? |
| :--- | :--- | :--- | :--- |
| **Estrategia de Activos** | Custodia lo que recibe (ETH, LINK, etc). | **Auto-Swap a Stablecoin (USDC).** | Elimina el riesgo de volatilidad para el protocolo. 1 USDC siempre vale 1 USDC, simplificando la solvencia del banco. |
| **Contabilidad** | Compleja (Mapping anidado por token). | **Unificada (Solo USDC).** | Reduce drásticamente el costo de gas de almacenamiento y elimina la necesidad de oráculos externos para calcular el TVL total. |
| **Accesibilidad** | Solo tokens whitelisted por el Admin. | **Cualquier token con liquidez.** | Aprovecha la "Composabilidad". Si el token tiene un par en Uniswap, el banco puede aceptarlo sin permiso previo (Permissionless). |
| **Seguridad de Valor** | Aproximada (Oráculos Chainlink). | **Exacta (Realized Value).** | En lugar de "estimar" cuánto vale un depósito, el banco "realiza" el valor al venderlo por USDC. El `BankCap` es preciso al centavo. |

---

## ⚙️ Arquitectura y Flujo de Datos

1.  **Input:** Usuario envía ETH o Token X + `minUsdcOut` (protección slippage).
2.  **Routing:** El contrato interactúa con el **Router de Uniswap V2**.
3.  **Swap:** Se ejecuta `swapExactTokensForTokens` o `swapExactETHForTokens`.
4.  **Settlement:** El contrato recibe USDC.
5.  **Accounting:** Se verifica el `BankCap` y se acredita el saldo interno del usuario.

---

## 📦 Instrucciones de Despliegue

### Prerrequisitos (Sepolia Testnet)
Necesitas las direcciones de los contratos de infraestructura DeFi existentes en la red.
* **Uniswap V2 Router:** `0xC532a74256D3Db42D0Bf7a0400fEFDbad7694008` (O el router oficial de tu entorno).
* **USDC Token:** `0x1c7D4B196Cb0C7B01d743Fbc6116a902379C7238` (O la dirección del token stablecoin).

### Parámetros del Constructor
1.  **`_bankCapUSDC`**: Límite máximo del banco (Cuidado con los decimales de USDC, suelen ser 6).
    * Ejemplo para $10,000 USD: `10000000000` (10,000 + 6 ceros).
2.  **`_router`**: Dirección del Router de Uniswap.
3.  **`_usdc`**: Dirección del contrato del token USDC.

---

## 💻 Guía de Interacción

### 1. Depositar Tokens (ERC-20) - "Zap In"
El usuario quiere depositar `LINK`, pero tener saldo en `USDC`.
1.  **Approve:** En el contrato de LINK, aprueba a KipuBankV3 para gastar tus tokens.
2.  **Deposit:** Llama a `depositTokenWithSwap`:
    * `tokenInput`: Dirección de LINK.
    * `amountIn`: Cuánto LINK depositas.
    * `minUsdcOut`: **Crucial.** La cantidad mínima de USDC que aceptas recibir. (Calcula esto off-chain para evitar *Slippage* alto).

### 2. Depositar ETH Nativo
1.  Llama a `depositETHWithSwap` enviando ETH en el campo `msg.value`.
2.  Define `minUsdcOut` para protegerte de fluctuaciones de precio durante la transacción.

### 3. Retirar
Como todos los activos se convirtieron al entrar, el retiro es simple:
* Llama a `withdrawUSDC(amount)`. Recibirás USDC directamente en tu wallet.

---

## 🧠 Decisiones de Diseño y Trade-offs

### 1. Slippage y Protección MEV
* **Decisión:** Requerir el parámetro `minUsdcOut` en los depósitos.
* **Trade-off:** Aumenta la fricción para el usuario (el frontend debe calcularlo), pero es **obligatorio** por seguridad. Sin esto, un bot (Sandwich Attack) podría ver tu transacción de depósito, manipular el precio en Uniswap, y hacer que tu depósito de 1 ETH se convierta en 0.01 USDC.

### 2. Unificación a USDC
* **Decisión:** Convertir todo obligatoriamente a Stablecoin.
* **Trade-off:** El usuario pierde la exposición al alza de sus activos ("Upside"). Si depositas ETH y el ETH sube un 50%, tu saldo en el banco sigue siendo el mismo en dólares.
* **Justificación:** KipuBankV3 prioriza la **preservación de capital** y la estabilidad sobre la especulación.

### 3. Dependencia de Uniswap V2
* **Decisión:** Usar la interfaz `IUniswapV2Router02`.
* **Trade-off:** Uniswap V3 ofrece mejor ejecución de precios, pero su integración es mucho más compleja (NFT positions, ticks). V2 es robusto, estándar y suficiente para garantizar la liquidez en este diseño educativo.

---

**Autor:** Neddy Etman Choque Flores
**Estado:** Implementación Académica (No Auditada)
