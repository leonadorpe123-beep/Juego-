# Rediseño Realista del Garaje de Boxes F1 y Optimización del Circuito 1

Plan de arquitectura y modelado para transformar el interior del garaje de boxes del Circuito 1 en un entorno de Fórmula 1 hiperrealista contemporáneo (fibra de carbono, grafito y acero inoxidable), eliminando elementos geométricos primitivos o estilo "cartoon", suprimiendo grúas y equipamiento duplicado y optimizando los focos del circuito mediante `InstancedMesh`.

## Decisiones Críticas y Preferencias Confirmadas

> [!IMPORTANT]
> Decisiones confirmadas por el usuario en la fase de clarificación:
> - **Estética del Garaje**: F1 Contemporáneo de alta tecnología basado en fibra de carbono oscura, grafito y acero inoxidable cepillado, sustituyendo cualquier forma primaria saturada o aspecto de juguete.
> - **Consolidación de Geometría y Rendimiento**:
>   - Eliminación de la llamada duplicada `buildPitEquipment()` en `TrackBuilder.ts` (que generaba cajas azules y amarillas redundantes).
>   - Fusión y unificación de las grúas aéreas y umbilicales en `CurvedPitBuildingBuilder.ts`.
>   - Reemplazo de las 360 clonaciones de `BufferGeometry` en las 24 torres de iluminación de `buildHighMastFloodlights()` por `InstancedMesh` en GPU.

---

## 1. Visión General del Rediseño

El garaje de boxes actual presenta cubos monocromáticos planos, colores primarios plásticos (azul y amarillo saturados) y cilindros genéricos que desentonan con el realismo visual del resto del simulador. El nuevo diseño convertirá los 7 garajes del pit lane en bahías de ingeniería de élite inspiradas en el paddock actual de la Fórmula 1.

```
┌────────────────────────────────────────────────────────────────────────┐
│                   CIRCUIT 1 PIT LANE & GARAGE SYSTEM                   │
├────────────────────────────────────────────────────────────────────────┤
│                                                                        │
│   [ CurvedPitBuildingBuilder ]                                         │
│   │                                                                    │
│   ├──► [ Garaje Interior F1 Hiperrealista ]                            │
│   │    ├── Suelo Epoxi Técnico Pulido con Guías de Entrada             │
│   │    ├── Muro de Fibra de Carbono con Paneles Acústicos Modulares    │
│   │    ├── Estaciones de Telemetría Multi-Pantalla con Gráficas Reales │
│   │    ├── Carros de Herramientas en Acero Inoxidable y Cajoneras      │
│   │    ├── Racks de Neumáticos con Mantas Térmicas de Kevlar Oscuro    │
│   │    └── Gatos Hidráulicos Rápidos Delanteros/Traseros de Titanio    │
│   │                                                                    │
│   └──► [ Grúas Aéreas y Brazos Neumáticos Unificados ] (Zero Duplicate)│
│                                                                        │
│   [ TrackBuilder Optimizado ]                                          │
│   │                                                                    │
│   ├──► Supresión de `buildPitEquipment()` duplicado                    │
│   └──► [ HighMastFloodlights con InstancedMesh ]                       │
│        (Mástil, cruceta y focos batched en 2 Draw Calls para 24 torres)│
└────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Especificación Detallada del Interior del Garaje

### A. Materiales PBR y Acabados Industriales
* **Fibra de carbono mate/satinada**: Textura de trama cruzada de composite para mamparas divisorias, mesas de ingeniería y detalles de equipamiento.
* **Acero inoxidable cepillado y titanio**: Carros de herramientas modulares tipo taller de élite (Snap-On / Beta motorsport), gatos de elevación y soportes mecánicos.
* **Resina epoxi pulida de alta resistencia**: Suelo con sutil reflejo especular, juntas de dilatación y líneas técnicas de parada en pit box.
* **Mantas térmicas Cordura/Kevlar**: Fundas térmicas texturizadas oscuras con micro-indicadores de temperatura y llantas forjadas con tuerca de buje monobloque.

### B. Elementos Interiores del Garaje
1. **Centro de Telemetría de Ingeniería**:
   * Consola ergonómica curvada con monitores LCD multi-panel de alta resolución.
   * Texturas dinámicas horneadas en Canvas con gráficas auténticas de F1: telemetría de suspensiones, mapa de calor de las 4 ruedas, curvas de velocidad y estado de batería ERS.
2. **Bahías de Herramientas y Apoyo Técnico**:
   * Armarios modulares metálicos con cajones en relieve, encimeras de acero inoxidable y soportes para pistolas de impacto.
   * Botellas de nitrógeno presurizado y mangueras en espiral de poliuretano industrial.
3. **Equipamiento de Pit Stop en Bahía**:
   * Gato hidráulico frontal de accionamiento rápido con palanca de pivote y ruedas de nylon técnico.
   * Carretillas móviles para conjuntos de neumáticos blandos, medios y duros debidamente rotulados sin colores chillones.
4. **Iluminación Quirúrgica LED**:
   * Regletas de luz cenital neutra 5000K integradas en el techo técnico para resaltar los contornos mecánicos y eliminar sombras planas.

---

## 3. Optimizaciones de Rendimiento y Eliminación de Duplicados

### A. Eliminación de `buildPitEquipment()` Duplicado
* En `TrackBuilder.ts` se eliminará la llamada y método `buildPitEquipment()`.
* Las grúas aéreas quedarán centralizadas de forma única y con geometría fluida en `CurvedPitBuildingBuilder.ts`, eliminando los 6 bloques azules y amarillos discordantes y ahorrando pases de renderizado.

### B. Instanciación de Torres de Focos (`HighMastFloodlights`)
* Actualmente, 24 torres de iluminación clonan geometrías individuales (`baseMastGeo.clone()`, `baseSpotHousingGeo.clone()`, etc.) y aplican transformaciones en la CPU, lo que genera más de 360 asignaciones de memoria en cada carga.
* Se transformará a `InstancedMesh`:
  * 1 `InstancedMesh` para todos los mástiles y brazos (24 instancias).
  * 1 `InstancedMesh` para los focos y carcasas (144 instancias).
  * 1 `InstancedMesh` para los lentes emisivos (144 instancias).
* **Resultado**: Reducción drástica del tiempo de inicialización del circuito y rebaja de draw calls.

---

## 4. Fases de Ejecución

1. **Rediseñar `CurvedPitBuildingBuilder.ts`**:
   * Implementar la nueva arquitectura interior del garaje con fibra de carbono, acero inoxidable y telemetría de competición.
   * Integrar texturas PBR generadas proceduralmente para monitores y mantas térmicas.
   * Refinar los brazos aéreos y mangueras con acabados de titanio y aluminio aeronáutico.
2. **Optimizar `TrackBuilder.ts`**:
   * Suprimir el método redundante `buildPitEquipment()` y su llamada en el constructor.
   * Migrar `buildHighMastFloodlights()` a `InstancedMesh`.
3. **Verificación y Rendimiento**:
   * Comprobar compilación mediante `lint_applet` y `compile_applet`.
   * Verificar estabilidad a 60/120 FPS sin stutters ni mallas duplicadas.
