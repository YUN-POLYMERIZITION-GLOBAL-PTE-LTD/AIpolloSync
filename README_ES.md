<div align="center">

# AIpolloSync: Servidor Multimedia P2P Remoto y Skill de Streaming para Hermes

[English](README.md) | [简体中文](README_ZH.md) | [日本語](README_JA.md) | [Deutsch](README_DE.md) | [Español](README_ES.md)

</div>

---

## 📖 Descripción General y Valor Principal

**AIpolloSync** es un servicio de intercambio remoto de archivos multimedia personales. Inicia un servidor multimedia local Flask + WebRTC, establece un túnel FRP de salida para acceso a Internet público y expone sus archivos de video locales a través de una interfaz de agente de IA integrada con WhatsApp.

**Valor principal**: Acceda a su biblioteca de archivos multimedia remotos en cualquier momento y lugar a través de WhatsApp conectado a Hermes. Las listas de reproducción multimedia son compatibles con el reproductor AIpollo.

---

## ⚙️ Requisitos Previos

* **Hermes**: Asegúrate de tener Hermes instalado y en ejecución localmente.
* **Excepción de Firewall / Antivirus**: Agrega una regla de confianza para `frpc.exe` en Windows Defender o tu software de seguridad.

---

## 🚀 Guía de Instalación y Uso

1. **Instalar**: Descarga e instala el skill `AIpolloSync` a través de HermesHub.
2. **Configura la Carpeta de Videos**: Crea una carpeta llamada `videos` dentro del directorio del skill y coloca los archivos MP4 que deseas reproducir.
3. **Conecta WhatsApp**: Configura e integra tu canal de WhatsApp en Hermes.
4. **Ejecuta el Skill**: Inicia el skill en Hermes.
5. **Interactúa (Dirigido por LLM)**: Chatea con la IA de forma natural en tu canal. Por ejemplo:
   - *"Muéstrame mi lista de videos."*
   - *"¿Tengo alguna película para ver?"*
   - *"Reproduce el video del gato."*
6. **Reproducción**: Haz clic en el enlace generado en la respuesta para reproducir tu video.

---

## 🔒 Divulgación de Seguridad y Red

### Crítico: El túnel FRP expone los servicios locales a Internet público

Este skill **automáticamente** descarga y ejecuta el **cliente FRP (Fast Reverse Proxy) (`frpc`)** al iniciarse. El binario `frpc` se obtiene de GitHub Releases y establece un túnel de salida hacia un servidor FRP remoto (`129.213.174.213:7000`), que a su vez expone su servicio de medios local (puerto 8000) a **Internet público** a través de un subdominio `*.yunfrp.net`.

**Esto amplía materialmente su superficie de ataque.** Cualquiera que conozca o descubra el subdominio público puede intentar acceder a sus archivos multimedia y al servicio Flask en su máquina.

### 1. Comportamiento automático del túnel (Sin confirmación del usuario)

- **Automático al iniciar**: El túnel FRP se inicia automáticamente cuando se ejecuta `scripts/media_server_flask.py`. No hay aviso, confirmación ni variable de entorno de control.
- **Descarga del binario**: En la primera ejecución, `frpc.exe` se descarga silenciosamente desde GitHub (`fatedier/frp` releases). Se requiere acceso a Internet.
- **Sin cambios en el firewall**: El túnel es solo de salida; no es necesario abrir puertos de entrada en su firewall.

### 2. Riesgo de cadena de suministro: Ejecución de binarios descargados

- El skill descarga y ejecuta un binario nativo (`frpc.exe`) desde GitHub Releases. El compromiso del repositorio de GitHub, del artefacto de lanzamiento o del transporte de red (MITM) podría resultar en **ejecución de código arbitrario** en su equipo con los mismos privilegios que el proceso Python.
- **Verificación de suma SHA256**: El código incluye sumas SHA256 codificadas tanto para el archivo zip como para el binario `frpc.exe` extraído (versión `0.65.0`). La descarga se rechaza si alguna suma no coincide. Esto protege contra manipulaciones de transporte y descargas corruptas, pero **no protege contra un compromiso del repositorio o lanzamiento de GitHub**.
- **Versión bloqueada**: La versión de FRP está fijada en `0.65.0`. Una actualización requiere un cambio de código y re-verificación SHA256. Esto evita actualizaciones silenciosas a versiones potencialmente comprometidas.

### 3. Estado de autenticación

- **Sin autenticación implementada**: El servidor Flask actualmente **no tiene autenticación HTTP Basic, mecanismo de token ni control de acceso**. Todas las rutas API y endpoints multimedia son públicamente accesibles para cualquiera que llegue al servidor, ya sea a través de LAN o del túnel FRP.
- **Riesgo**: Un tercero no autenticado que descubra el subdominio `*.yunfrp.net` puede enumerar y descargar archivos multimedia de su máquina.

### 4. Confianza en el servidor remoto

- El servidor FRP en `129.213.174.213:7000` es un relay de terceros. Todo el tráfico entre Internet público y su servicio local pasa a través de este servidor.
- El túnel FRP opera en modo HTTP (sin terminación TLS por parte del servidor FRP).
- Debe confiar en que el operador de este servidor FRP no inspeccionará, registrará ni manipulará su tráfico.

---

## 🛡️ Recomendaciones de Seguridad

* **Servidor Dedicado / Máquina Virtual**: Para una seguridad óptima, se recomienda ejecutar este servicio en un dispositivo secundario o dentro de una Máquina Virtual (VM) aislada.
* **Mantenimiento Periódico**: Mantén actualizados tu sistema operativo y el entorno Hermes con los últimos parches de seguridad.

---

## 💻 Compatibilidad de Plataforma

* **Soporte Actual**: Windows (x64)
* **Próximamente**: Soporte para Linux y macOS en desarrollo.

*Si necesitas soporte para otras plataformas o tienes dudas, abre un Issue en GitHub. ¡Gracias por tu confianza y apoyo!*