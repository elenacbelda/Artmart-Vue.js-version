# ArtMart

ArtMart es una plataforma e-commerce desarrollada como una SPA (Single Page Application) enfocada en la exploración, personalización y gestión de obras de arte. El proyecto integra un configurador avanzado que permite personalizar las dimensiones y acabados de los marcos, además de una funcionalidad colaborativa en tiempo real.

## Características Principales

* **Configurador Inteligente:** Interfaz reactiva que permite personalizar al milímetro el marco de cada obra, sincronizando visualmente el slider con los inputs numéricos.
* **Colaboración en Tiempo Real:** Implementación del protocolo de WebSockets para sesiones "Configure Together", permitiendo a varios usuarios participar simultáneamente en la configuración de una obra.
* **Gestión de Estado Centralizada:** Uso de Pinia para mantener la consistencia del carrito, el cálculo dinámico de precios y el estado de la sesión en toda la aplicación.
* **Experiencia de Usuario (UX):** Flujo de checkout optimizado con gestión de estados asíncronos y diseño responsivo adaptado a múltiples dispositivos.

## Stack Tecnológico

* **Frontend:** Vue 3 (Options API)
* **Gestión de Estado:** Pinia
* **Enrutamiento:** Vue Router
* **Comunicación:** REST API & WebSockets
* **Tooling:** Vite

## Arquitectura y Retos Técnicos

Este proyecto ha sido desarrollado siguiendo una arquitectura modular basada en componentes. Los retos técnicos principales incluyen:

1.  **Sincronización Bidireccional:** Gestión eficiente de estados mediante `v-model` personalizado en componentes complejos para asegurar la integridad de los datos de configuración.
2.  **Ciclo de Vida de WebSockets:** Manejo robusto de la conexión entre host y invitados, incluyendo la resolución de conflictos, notificaciones de estado y desconexiones automáticas.
3.  **Integración con Backend Desacoplado:** Consumo de servicios RESTful mediante una capa de abstracción, garantizando que la lógica de negocio y la de UI estén claramente separadas.
