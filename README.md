# 🤖 BEVi: Sistema Automatizado de Procesamiento Tributario con n8n e IA

### 🗺️ Diagrama de Arquitectura de BEVi

```mermaid
---
---
config:
  layout: fixed
  look: classic
---
flowchart LR
 subgraph Fase1["Fase 1: Ingesta (Cartero)"]
        B("⚙️ n8n: Separar Adjuntos")
        A["📧 Trigger: Llega Correo con Factura"]
        C[("📂 Drive: Guardar en Inbox temporal")]
  end
 subgraph Fase2["Fase 2: Enrutamiento (Archivador)"]
        D["⚙️ Trigger: Nuevo Archivo en Inbox"]
        E{"🧠 Gemini 2.5 Pro: Extraer RUT"}
        F[("📊 Sheets: Consultar Directorio")]
        G{"¿Proveedor Existe?"}
        H["📂 Drive: Mover a Carpeta Existente"]
        I["📂 Drive: Crear Nueva Carpeta"]
        J[("📊 Sheets: Actualizar Directorio")]
        K["📂 Drive: Mover a Nueva Carpeta"]
        L(("Activar Fase 3"))
  end
 subgraph Fase3["Fase 3: Motor Matemático y Output"]
        M{"🧠 Gemini 2.5 Pro: Extraer Líneas de Factura"}
        N("⚙️ JS: Motor Matemático - Fletes e ILA")
        O("📊 Generar Excel Limpio")
        P[("📂 Drive: Guardar Excel del Proveedor")]
        Q["📧 Gmail: Enviar Excel al Remitente"]
  end
    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G -- Sí --> H
    G -- No --> I
    I --> J
    J --> K
    H --> L
    K --> L
    L --> M
    M --> N
    N --> O
    O --> P & Q

    %% --- BLOQUE DE ESTILOS CORREGIDO ---
    
    %% Nodos oscuros con texto BLANCO (Gmail y Gemini)
    style A fill:#d93025,stroke:#b31412,color:#fff
    style Q fill:#d93025,stroke:#b31412,color:#fff
    style E fill:#8e24aa,stroke:#6a1b9a,color:#fff
    style M fill:#8e24aa,stroke:#6a1b9a,color:#fff

    %% Nodos claros con texto NEGRO (Drive, Sheets, n8n, JS)
    style B fill:#f5f5f5,stroke:#bdbdbd,color:#000
    style C fill:#cce0ff,stroke:#1a73e8,color:#000
    style D fill:#f5f5f5,stroke:#bdbdbd,color:#000
    style F fill:#ceead6,stroke:#188038,color:#000
    style H fill:#cce0ff,stroke:#1a73e8,color:#000
    style I fill:#cce0ff,stroke:#1a73e8,color:#000
    style J fill:#ceead6,stroke:#188038,color:#000
    style K fill:#cce0ff,stroke:#1a73e8,color:#000
    style N fill:#f5f5f5,stroke:#bdbdbd,color:#000
    style O fill:#f5f5f5,stroke:#bdbdbd,color:#000
    style P fill:#cce0ff,stroke:#1a73e8,color:#000

## 📖 El Origen del Proyecto
Si bien la arquitectura de este sistema nace tras analizar las ineficiencias crónicas en la cadena de suministro del **sector gastronómico**, **BEVi** fue desarrollado originalmente como una **solución a medida para un cliente específico** del rubro. 

El cliente enfrentaba un cuello de botella crítico en sus operaciones: necesitaba transformar facturas tributarias caóticas en un **formato tabular estricto**, extrayendo la información línea por línea. Más importante aún, requería un motor automatizado capaz de calcular el **costo real y exacto** de cada producto comprado, integrando en el valor unitario tanto la tributación específica (impuestos de alcoholes y bebidas) como el prorrateo de servicios adjuntos (fletes y logística).

### ⚠️ Los Retos a Superar
Para cumplir con este requerimiento, BEVi debía sortear dos grandes fricciones operativas:

**1. La Barrera de la No-Estandarización:** El cliente interactuaba con decenas de proveedores que enviaban documentos en formatos radicalmente distintos (PDFs nativos, escaneos torcidos o fotografías). Además, las descripciones, códigos y unidades de medida (cajas vs. botellas) no seguían ningún estándar. Extraer esta data línea por línea mediante sistemas OCR tradicionales o reglas fijas era imposible, lo que obligaba a realizar un *data entry* manual lento y propenso a errores.

**2. Complejidad Tributaria y Costos Ocultos:** En Chile, los productos de alta rotación (licores, cervezas, bebidas azucaradas) están sujetos a impuestos adicionales al IVA (ILA/IABA), con tasas que varían drásticamente (del 10% al 31.5%). Las facturas rara vez desglosan estos impuestos de forma amigable, y a menudo incluyen cobros globales por despacho. El desafío matemático era lograr que la Inteligencia Artificial identificara el tipo de producto, aplicara la tasa tributaria correcta y distribuyera el flete global proporcionalmente en cada línea, para así obtener el costo unitario real.

### 💡 La Solución Técnica
**BEVi** es un orquestador automatizado que actúa como un analista de datos autónomo. Utiliza **n8n** para la logística de archivos y **Google Gemini 2.5 Pro (Visión y LLM)** para leer e interpretar los documentos no estructurados con alta precisión. 

Una vez que la IA extrae la información en crudo, los datos pasan por el **motor matemático personalizado de BEVi** (construido en JavaScript) que:
* Identifica y normaliza las cantidades por línea de producto.
* Infiere o extrae los impuestos específicos (ILA) excluyendo el IVA.
* Prorratea los costos logísticos en función del valor neto.
* Calcula matemáticamente el **Costo Unitario Real**.

El resultado final es una matriz plana exportada en Excel `.xlsx`, entregando el formato tabular exacto que el cliente requería, listo para ser ingestado en su ERP o sistema de inventario sin intervención humana.

---

## 🛠️ Stack Tecnológico
* **Orquestador:** n8n (Node-based workflow automation)
* **Inteligencia Artificial:** Google Gemini 2.5 Pro (Extracción OCR avanzada y parsing JSON estructurado)
* **Almacenamiento y Archivos:** Google Drive API
* **Base de Datos Dinámica:** Google Sheets API
* **Comunicaciones:** Gmail API (Trigger y despachador de correos)
* **Procesamiento:** JavaScript (Motores matemáticos, expresiones regulares y limpieza de strings)

---

## ⚙️ Arquitectura y Flujo de Datos

El cerebro de BEVi está dividido en tres flujos de trabajo (*workflows*) secuenciales y autónomos:

### Fase 1: Ingesta (WF 00 - Cartero 📬)
Actúa como el punto de entrada de BEVi. 
* Un nodo *Trigger* de Gmail escucha continuamente el buzón de recepción.
* Un script de JavaScript aísla los archivos adjuntos (binarios) de cada correo y extrae el correo electrónico del remitente.
* Los documentos se guardan temporalmente en una carpeta de "Inbox" en Google Drive, inyectando el correo del remitente en los metadatos del archivo para no perder la trazabilidad.

### Fase 2: Enrutamiento y Directorio (WF 01 - Inbox Archivador 📁)
Se encarga de la logística de carpetas y base de datos de proveedores.
* BEVi detecta un nuevo archivo en el Inbox y lo descarga.
* Un modelo Gemini ultraligero extrae únicamente el RUT y el Nombre Comercial del proveedor desde la cabecera del documento.
* Un nodo de código formatea matemáticamente el RUT (ej. `XX.XXX.XXX-X`).
* Se consulta el Directorio Maestro en Google Sheets. 
  * **Si es un proveedor nuevo:** Crea una nueva carpeta en Drive con su nombre, guarda la URL de la carpeta en el Directorio y mueve el archivo.
  * **Si ya existe:** Mueve el archivo directamente a la carpeta histórica del proveedor.
* Lanza el archivo al tercer flujo de trabajo.

### Fase 3: Motor Matemático y Output (WF 02 - Extractor de Datos 🤖)
El corazón analítico del proyecto.
* Gemini 2.5 Pro lee la factura completa (PDF o imagen) y extrae cada línea de producto, cantidades, impuestos específicos y descuentos en un JSON estructurado, ignorando el IVA para evitar conflictos.
* **El Motor Matemático (JavaScript):** Toma el JSON crudo y ejecuta la lógica de negocios:
  * Prorratea el flete global entre todas las unidades de forma proporcional.
  * Calcula el Impuesto a los Alcoholes y Bebidas Analcohólicas (ILA/IABA).
  * Calcula el **Costo Unitario Real** (Neto + ILA + Flete / Cantidad) para decisiones precisas de compra.
* Genera una matriz plana de 9 columnas clave y la exporta como un archivo Excel `.xlsx`.
* El Excel se guarda en la carpeta del proveedor en Drive y BEVi lo envía automáticamente por correo como respuesta a quien solicitó el procesamiento.

---

## 🚀 Instalación y Despliegue

1. **Requisitos Previos:**
   * Una instancia de n8n activa.
   * Proyecto en Google Cloud Console con las APIs habilitadas (Gmail, Drive, Sheets) y credenciales OAuth2.
   * API Key de Google AI Studio (Gemini).
   * Un archivo de Google Sheets con las columnas: `Rut_proveedor`, `Nombre_Comercial`, y `Folder_ID`.
2. **Importación:**
   * Descarga los 3 archivos `.json` de este repositorio.
   * En tu entorno de n8n, crea tres workflows nuevos y utiliza la opción *Import from File*.
3. **Configuración:**
   * Conecta tus credenciales de Google y Gemini en los nodos correspondientes.
   * Actualiza los `Folder ID` en los nodos de Google Drive apuntando a las carpetas raíz de tu propio ecosistema de Drive.

---

## 🗺️ Roadmap y Próximos Pasos (Mejoras a Futuro)

BEVi está diseñado bajo una arquitectura escalable. Las siguientes mejoras están planificadas para las próximas versiones (v2.0):

* **1. Enrutamiento Multi-Sucursal (Sistema Alias/Forwarding):** Implementación de dominios personalizados con direcciones dedicadas por sucursal (ej. `providencia@midominio.com`). Los correos tendrán reglas de *auto-forwarding* al Cartero central, permitiendo etiquetar el origen de cada factura. Esto habilitará una reestructuración profunda en Google Drive, separando los archivos bajo una jerarquía estricta: `Sucursal -> (Inputs / Outputs) -> Proveedor`.
* **2. Data Warehouse Centralizado (Google BigQuery):**
  Migración del almacenamiento de datos desde Google Sheets hacia Google BigQuery. Esta centralización permitirá crear tablas relacionales robustas para cruzar y comparar la evolución de los costos unitarios de un mismo producto a través de distintos proveedores, alimentando Dashboards de Inteligencia de Negocios en tiempo real.
* **3. Consolidación Automatizada (CSV Batching):**
  Desarrollo de un micro-flujo complementario capaz de tomar múltiples outputs seleccionados y combinarlos automáticamente en un archivo `CSV` maestro estandarizado, dejándolo formateado y listo para ser ingestado de forma masiva en sistemas ERP o bases de datos de inventario.