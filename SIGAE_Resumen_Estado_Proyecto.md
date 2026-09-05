# SIGAE — Resumen de Estado del Proyecto (para continuar en cuenta nueva)

**Fecha de este resumen:** generado para transición a Claude Pro. Úsalo como primer mensaje al retomar, junto con los archivos `.md` que ya tienes descargados.

---

## 1. Qué es SIGAE

Sistema de Inspección y Gestión de Activos Eléctricos: app móvil (React Native + Expo SDK 57 + TypeScript) para que técnicos de campo inventarien instalaciones eléctricas industriales — zonas, activos, componentes, y su topología eléctrica completa (qué alimenta a qué, hasta el nivel de breaker).

**Norte del proyecto:** que el cliente pueda armar el diagrama unifilar de un tablero (conexiones internas y externas), administrarlo desde la app, y exportarlo más adelante a AutoCAD vía Python (`ezdxf`). Cualquier propuesta nueva se evalúa contra esto.

**Entorno de desarrollo:** Google Antigravity (IDE agéntico). Se prueba en Expo Go, en dispositivo Android real y en Web (`--web`), aunque se descubrió que confirmar solo en Web no garantiza que funcione en móvil real — regla vigente: **toda confirmación de checkpoint debe indicar la plataforma probada**.

---

## 2. Decisiones técnicas clave ya tomadas (no reabrir sin razón)

- **Mapas:** Leaflet + OpenStreetMap dentro de un `WebView` (`react-native-webview`) — no `react-native-maps`/Google Maps, para evitar API keys/facturación y permitir caché offline futura de tiles.
- **IDs:** siempre UUID vía `expo-crypto` (`Crypto.randomUUID()`). Nunca `Date.now()`, índices ni incrementales. Estables durante edición/eliminación/sync futura.
- **Arquitectura:** UI → Services → Repository → AsyncStorage (provisional; migrará a SQLite en la fase Offline).
- **Editor de diagramas:** `react-native-svg` + `react-native-gesture-handler` + `react-native-reanimated` (nativo, no WebView).
- **Modelo de datos eléctrico:** Nodo → Terminal → Conexión (no solo Nodo→Nodo). Cada Nodo (Subestación, Transformador, Tablero, Barraje, Breaker, Contactor, Relé Térmico, Motor, MCC, CCM, UPS, Generador, Alimentador, Circuito, Otro) tiene Terminales de tipo Potencia o Mando.
- **Representación UNIFILAR, no trifilar:** una sola línea por conexión de potencia entre dispositivos (no una línea por fase). Los datos de fases/neutro/tierra/calibre/material/tensión/longitud son *atributos* de la Conexión, mostrados como etiqueta configurable (Básica/Técnica/Completa), no como líneas separadas. Terminales de Potencia se redujeron a 1 entrada + 1 salida por dispositivo (polos como atributo del Nodo, no como cantidad de terminales).
- **Símbolos:** SVG propios (no descargados de terceros por riesgo de licencia), formas geométricas simples pero distintivas por tipo, en `src/assets/symbols/iec/` y `ansi/`, con toggle de estándar.
- **Principio de usabilidad:** técnicos de campo son mayores, reacios a formularios complejos. Toda creación debe pedir el mínimo posible (idealmente solo "Nombre"), con biblioteca visual de símbolos en vez de listas de texto, y detalles técnicos avanzados escondidos tras "Ver más".

---

## 3. Qué está construido y verificado

| Módulo | Estado |
|---|---|
| Arquitectura base (Tarea 0) | ✅ |
| Zonas: CRUD, foto (cámara real), GPS/mapa Leaflet, edición | ✅ |
| Navegación: Stack raíz + Tabs (Inicio/Zonas/Activos/＋), Inicio minimalista | ✅ |
| Catálogos de Tipos (Activo y Componente) con campos característicos, duplicar, ~25 datos semilla cada uno | ✅ |
| Activos: CRUD, catálogo conectado, duplicar, "Sin dato" | ✅ |
| Vista GIS de Activos + extensión estilo Map Marker (íconos por tipo, clustering, búsqueda, compartir, agregar desde mapa) | ✅ (pero ver bug abierto abajo) |
| Modelo Nodo→Terminal→Conexión + validaciones (entrada única, sin ciclos, matriz tipo-a-tipo, Potencia≠Mando) | ✅ |
| Editor visual de Diagrama Eléctrico: lienzo con pan/zoom por gestos, arrastre de nodos, buscador/vista de lista para diagramas densos | ✅ |
| Símbolos SVG IEC/ANSI (15 tipos) | ✅ |
| Modo Conectar (tocar terminal→terminal, conexión externa entre Activos vía selector Zona→Activo→Nodo→Terminal) | ✅ (pero ver bug abierto abajo) |
| Modelo unifilar (1 línea por conexión + atributos técnicos + etiqueta configurable) | ✅ |
| Plantilla "Arranque Directo de Motor" (ITM→Contactor→Relé Térmico→Motor, basada en normas reales documentadas por el usuario) | ✅ |
| Fase D: navegación cruzada Mapa↔Diagrama | ⚠️ Implementada pero bloqueada por el bug de Vista GIS (no se puede verificar visualmente) |
| Evidencias por Activo (5 categorías de foto) | ✅ Verificado en móvil real |
| Biblioteca Técnica (documentos PDF/planos por Activo) | ✅ Verificado en móvil real |

---

## 4. 🔴 Bug abierto — bloqueante, es lo primero a resolver al retomar

**Dos problemas específicos de móvil real** (no aparecían en Web):

1. **Líneas de conexión no se ven** en el Diagrama Eléctrico en dispositivo Android.
2. **Vista GIS aparece en negro**, no carga el mapa.

Se envió un primer diagnóstico a Antigravity (hipótesis: `zIndex` de estilo que funciona en Web pero no en `react-native-svg` nativo, para las líneas; y problema de cómo se pasa el HTML al `WebView` en nativo vs. web para el mapa). **Antigravity aplicó una corrección pero el usuario reportó "sigue con problemas"** — sin detalle todavía de qué exactamente falla ahora ni qué dijo el reporte de Antigravity sobre la causa que identificó.

**Antes de avanzar con cualquier otra tarea, en la nueva sesión hay que:**
1. Pedirle al usuario el reporte que dio Antigravity tras la última corrección.
2. Pedirle síntomas actuales exactos (¿las líneas no aparecen en absoluto o aparecen mal ubicadas? ¿la Vista GIS sigue negra total o cambió algo?).
3. Pedir cualquier mensaje de error visible.
4. Recién con eso, dar una segunda corrección más precisa — no repetir el mismo diagnóstico genérico.

---

## 5. Pendiente inmediato después del bug (orden del roadmap)

1. Resolver el bug de arriba.
2. Retomar/confirmar checkpoint de **Fase D** (navegación Mapa↔Diagrama) una vez el mapa funcione.
3. **Offline/Sincronización**: migrar AsyncStorage → SQLite, cola de sync, resolución de conflictos (no last-write-wins automático — marcar para revisión de Supervisor).
4. **Optimización**: listas virtualizadas, compresión de imágenes, rendimiento con muchos activos.
5. **Exportación CAD**: servicio Python con `ezdxf`, el objetivo final del proyecto.

## 6. Movido al final del roadmap (decisión ya tomada, no reabrir el diseño)

- **Sistema de Plantillas Personalizables** (Opción B): plantilla genérica de Tablero con cantidad de breakers configurable + "Guardar como plantilla propia" para que el equipo arme su propia biblioteca. Requiere agregar `Barraje→Breaker` y `Tablero→Breaker` a la matriz de compatibilidad.
- **Catálogo de Equipos estilo EPLAN**: dispositivos reales de fabricante (ej. "Contactor Schneider LC1D12") con foto de referencia + specs de fábrica pre-cargadas (aquí sí es válido tener valores por defecto, porque son datos reales de catálogo, no suposiciones). El símbolo dibujado sigue siendo el estándar IEC/ANSI por tipo, no varía por marca — la foto es solo para reconocimiento en campo.

---

## 7. Archivos de referencia que deberías tener descargados

- `SIGAE_SRS_v1.1_Corregido.md` — especificación completa
- `SIGAE_Plan_Tareas_Antigravity.md` — **el más importante**, tabla de estado siempre actualizada
- `SIGAE_Logica_Conexiones_y_Exportacion_CAD.md` — diseño del modelo Nodo/Terminal/Conexión
- `SIGAE_Bug_Movil_Lineas_y_GIS_Negro.md` — diagnóstico del bug abierto (primera versión, insuficiente)

## 8. Cómo retomar en la cuenta nueva

Pega este archivo completo como primer mensaje, junto con el `SIGAE_Plan_Tareas_Antigravity.md`, y luego directamente el reporte de Antigravity + síntomas actuales del bug de la sección 4 — así se puede dar una corrección precisa sin repetir preguntas ya respondidas en esta conversación.
