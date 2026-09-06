# Plan: Aplicación para Organización de Apoyo Infantil

> **Estado:** Borrador para revisión
> **Fecha:** 2026-09-05
> **Repo objetivo:** Otro repo separado (este documento se migrará cuando exista).

---

## 1. Contexto

Una organización que apoya a niños necesita una aplicación propia para:

- Analizar la información emocional registrada por SENTIA.
- Dar seguimiento al progreso de cada niño.
- Detectar alertas o patrones de riesgo.
- Generar reportes exportables para profesionales.

**Importante:** la aplicación de la organización **NO vive en el repo de SENTIA**. SENTIA queda como la app de las familias; la nueva app es el espacio profesional del grupo de apoyo.

---

## 2. Objetivo

Construir un **panel profesional** donde el grupo de apoyo pueda:

1. Ver eventos emocionales registrados por cada niño.
2. Consultar perfiles y evolución/progreso.
3. Detectar patrones de riesgo (emociones negativas repetidas, cambios bruscos, horarios).
4. Exportar reportes claros para terapeutas o para la propia organización.

---

## 3. Usuarios y roles

### Usuarios objetivo

- Grupo de apoyo / profesionales (psicólogos, terapeutas, voluntarios autorizados).

### Modelo de acceso

- **Una sola organización** en el MVP.
- Todos los usuarios profesionales de la organización ven los mismos niños/datos.
- A futuro se puede escalar a varias organizaciones (multi-tenant), pero el diseño debe preverlo desde el inicio para no rehacer seguridad después.

### Roles sugeridos (fase inicial)

| Rol | Permisos |
|---|---|
| Administrador | Invitar miembros, gestionar acceso a niños, configurar alertas |
| Profesional | Ver niños asignados, analizar eventos, generar reportes |
| (Futuro) Supervisor | Ver datos agregados de toda la organización |

---

## 4. Plataformas objetivo

| Plataforma | Prioridad |
|---|---|
| Windows (computadora) | Alta — análisis de datos, reportes |
| Android (móvil/tablet) | Alta — consulta en campo/sesiones |
| Web (navegador) | Media — puede derivarse de Flutter web |
| iOS | Baja/después |

---

## 5. Stack recomendado

### Recomendación: Flutter (Dart)

- **Un solo código** para Windows desktop, Android y, más adelante, Web/iOS.
- SENTIA ya está en Flutter → se pueden copiar/adaptar modelos de emoción, colores y reglas de dominio.
- SDK oficial de Firebase/Firestore disponible.
- Buen soporte para dashboards responsivos con `fl_chart`, `DataTable`, etc.
- Es multiplataforma nativa: Windows y Android desde una misma base.

### Alternativas consideradas

| Opción | Ventaja | Desventaja |
|---|---|---|
| React/Next.js (PWA) | Muy bueno para dashboards web, fácil de desplegar | No es app nativa; funciona en navegador Windows/Android |
| Kotlin Multiplatform + Compose | Otro lenguaje moderno, nativo en Android/desktop | Curva de aprendizaje, menos maduro para dashboards que Flutter |
| .NET MAUI / Avalonia (C#) | Bueno en ecosistema Microsoft/Windows | Menos natural para Android + Firebase en este contexto |
| Flutter (Recomendado) | Una base, ya conocida en el proyecto, Android+Windows | Dart no es “otro lenguaje” si se buscaba expandir el stack |

**Conclusión:** si el objetivo es **Windows + Android**, lo más eficiente es **Flutter**. Si además quieres aprender/expandir a otro lenguaje, Kotlin Multiplatform es la alternativa más razonable, pero alarga el MVP.

---

## 6. Arquitectura propuesta

```
┌──────────────────────────────┐
│  App organización (Flutter)  │
│  Windows desktop             │
│  Android móvil/tablet        │
└──────────────┬───────────────┘
               │ Firebase Auth
               ▼
┌──────────────────────────────┐
│  Firebase / Firestore        │
│  Misma infraestructura SENTIA│
│  + colecciones profesionales │
└──────────────┬───────────────┘
               │
┌──────────────▼───────────────┐
│  Datos SENTIA existentes     │
│  users/{userId}/children     │
│  users/{userId}/emotion_events│
└──────────────────────────────┘
```

### Principios

- **SENTIA no se rompe:** la app profesional es solo lectura de los datos compartidos (excepto notas/alertas propias).
- **Consentimiento obligatorio:** una familia debe autorizar compartir el perfil/eventos con la organización antes de que el profesional pueda verlos.
- **Seguridad por defecto:** reglas de Firestore, roles y mínimo acceso.

---

## 7. Datos actuales de SENTIA (para copiar al plan técnico)

### Eventos emocionales

- Ubicación Firestore: `users/{userId}/emotion_events`
- Campos:
  - `emotion`: `anger`, `sadness`, `happiness`, `disgust`, `fear`, `surprise`
  - `timestamp`: Timestamp de Firestore

### Perfiles de niños

- Ubicación Firestore: `users/{userId}/children`
- Campos principales:
  - `name`
  - `age`
  - `gender` (opcional)
  - `birthDate` (opcional)
  - `favoriteColorValue` (opcional)
  - `favoriteToy` (opcional)
  - `routines` (lista)
  - `avatarUrl` (opcional)

### Reglas actuales

- `firestore.rules` solo permite que un usuario lea/escriba **sus propios datos**:

```
match /users/{userId}/{document=**} {
  allow read, write: if request.auth != null && request.auth.uid == userId;
}
```

Esto significa que **hoy la organización NO puede leer los datos de SENTIA**. Habrá que agregar un mecanismo de autorización/consentimiento.

---

## 8. Alcance MVP

### ✅ Debe tener (MVP)

1. **Login profesional**
   - Firebase Auth con correo/contraseña.
   - Solo miembros invitados de la organización pueden entrar.

2. **Lista de niños visibles**
   - Mostrar niños que han sido **compartidos/consentidos** con la organización.
   - Datos básicos: nombre, edad, avatar.

3. **Detalle de niño**
   - Perfil.
   - Historial de emociones.
   - Línea de tiempo / calendario.

4. **Dashboard de emociones**
   - Distribución por emoción (últimos 7/30/90 días).
   - Frecuencia por día/hora.
   - Tendencia semanal.

5. **Alertas / patrones de riesgo**
   - Regla base: N emociones negativas consecutivas en X días.
   - Cambio brusco vs. línea base del niño.
   - Lista de alertas activas.

6. **Reportes exportables**
   - Resumen por niño (PDF o CSV).
   - Reporte del periodo con gráficas y notas.

### 🟡 Debería tener (post-MVP)

- Notas profesionales por niño (solo visibles para la organización).
- Filtros avanzados por emoción, fecha, hora y rango de edad.
- Comparativa entre niños.
- Exportación PDF con diseño de la organización.

### 🔵 Sería bueno a futuro

- Multi-tenant (varias organizaciones).
- Envío de alertas por correo.
- Panel agregado/estadístico de la organización.
- App iOS.
- Roles más finos (supervisor, lector, editor).

---

## 9. Consentimiento y privacidad (crítico)

La app maneja datos de **menores**, así que esto no es opcional:

### Modelo propuesto

Agregar a Firestore una colección de autorización, por ejemplo:

```
consents/{consentId}
  userId: string
  childId: string
  organizationId: string
  status: "pending" | "active" | "revoked"
  createdAt: timestamp
  expiresAt?: timestamp
```

O bien, si se prefiere más simple:

- En el documento del niño agregar: `sharedOrganizations: ["orgId"]`

### Regla mental

> El profesional solo puede leer `users/{userId}/children/{childId}` y
> `users/{userId}/emotion_events` si existe un consentimiento activo entre
> esa familia, ese niño y la organización.

### Pregunta abierta para el grupo

- ¿Cómo obtienen el consentimiento? ¿La familia lo activa desde SENTIA, o la organización registra un documento firmado y un administrador lo habilita?

---

## 10. Fases del proyecto

### Fase 0 — Validación (1-2 semanas)

- Confirmar necesidades reales con el grupo.
- Definir quién invita/administra usuarios.
- Definir proceso de consentimiento.
- Crear repo nuevo y estructura base Flutter.

### Fase 1 — MVP (4-6 semanas)

- Login + roles básicos.
- Lista de niños compartidos.
- Detalle de niño + eventos emocionales.
- Dashboard simple (7/30 días).
- Reglas iniciales de riesgo.
- Exportación CSV (rápida) y PDF básico.
- Seguridad: reglas de Firestore + consentimiento.

### Fase 2 — Consolidación (3-4 semanas)

- Notas profesionales.
- Alertas con estados (nueva/resuelta/ignorada).
- Reportes PDF mejorados.
- Pruebas con el grupo real.

### Fase 3 — Escalamiento

- Multi-tenant si hay más organizaciones.
- Dashboard agregado.
- Automatización de alertas (correo/notificaciones).

---

## 11. Stack técnico detallado (si se elige Flutter)

| Capa | Tecnología sugerida |
|---|---|
| UI | Flutter (Material 3) |
| Estado | Riverpod (igual que SENTIA) |
| Navegación | go_router |
| Backend/BBDD | Firebase Auth + Cloud Firestore |
| Gráficas | fl_chart |
| Reportes | PDF: `pdf` + `printing`; CSV: exportación manual |
| Desktop | Flutter Windows |
| Android | Flutter Android (tablet/móvil) |

---

## 12. Riesgos

| Riesgo | Mitigación |
|---|---|
| Acceso indebido a datos de menores | Consentimiento + reglas estrictas + revisión de seguridad |
| Romper la app actual de SENTIA al tocar Firestore | No tocar colecciones actuales; agregar colecciones nuevas y reglas compatibles |
| El grupo no adopta la herramienta | Prototipo rápido con datos reales de prueba y feedback en Fase 2 |
| Volumen de eventos alto | Consultas con filtros de fecha, índices compuestos, paginación |
| Reportes PDF pesados/complejos | Empezar con CSV y PDF simple |
| Stack no alineado con el grupo | Decidir con ellos antes de codificar |

---

## 13. Decisiones pendientes

- [ ] Confirmar stack final: **Flutter** vs otra opción.
- [ ] Crear/definir ruta del repo nuevo.
- [ ] Definir flujo de consentimiento de familias.
- [ ] Definir quién administra usuarios de la organización.
- [ ] Confirmar si usará la misma cuenta Firebase de SENTIA o un proyecto nuevo con copia/exportación.
- [ ] Definir si el profesional necesita ver solo niños asignados a él o todos los de la organización.
- [ ] Definir formato exacto de reportes (qué incluye el PDF).
- [ ] Definir reglas de riesgo con el grupo (cantidad de eventos, ventana de tiempo, umbrales).

---

## 14. Siguiente paso sugerido

1. Leer este plan con el grupo de apoyo.
2. Marcar/responder las decisiones pendientes.
3. Crear el repo nuevo.
4. Arrancar Fase 0 con un prototipo de pantallas y confirmación del flujo de consentimiento.
