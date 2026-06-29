# Museo de La Infamia — Infra

## Qué es este proyecto
Configuración de infraestructura Firebase para "Museo de La Infamia - Terremotos Venezuela 2026". Contiene las reglas de seguridad de Firestore y Storage, los índices de Firestore, y la configuración de Firebase Hosting.

## Stack
- **Firebase Project ID**: `museo-infamia-vzla`
- **Firestore**: base de datos de denuncias
- **Firebase Storage**: almacenamiento de imágenes de evidencia
- **Firebase Hosting**: sirve el frontend React + proxying a Cloud Run
- **CI/CD**: GitHub Actions (`.github/workflows/deploy.yml`)
  - Rama `development` → deploy de rules al proyecto Firebase
  - Rama `production` → deploy de rules al proyecto Firebase

## Repos relacionados
| Repo | Propósito |
|---|---|
| `CarlosArdilaP/museo-infamia-frontend` | React frontend en Firebase Hosting |
| `CarlosArdilaP/museo-infamia-api` | NestJS 11 API en Cloud Run |

## Archivos
| Archivo | Propósito |
|---|---|
| `firestore.rules` | Reglas de seguridad Firestore |
| `firestore.indexes.json` | Índices compuestos para queries |
| `storage.rules` | Reglas de seguridad Storage |
| `firebase.json` | Config Hosting: rewrites, headers de seguridad |
| `.firebaserc` | Asocia el proyecto con `museo-infamia-vzla` |

## Reglas de Firestore (`firestore.rules`)
- **Lectura pública**: solo denuncias con `estado == 'aprobada'`
- **Escritura**: SIEMPRE denegada desde el cliente (solo Cloud Run con Admin SDK puede escribir)
- **Colección `/admins`**: solo el propio admin puede leer su documento

## Reglas de Storage (`storage.rules`)
- **Lectura**: pública (las imágenes de denuncias son públicas una vez aprobadas)
- **Escritura**: denegada desde el cliente (solo via signed URLs generadas por la API)

## Firebase Hosting (`firebase.json`)
- `/api/**` → proxy a Cloud Run service `museo-infamia-api` (región `us-central1`)
- `**` → SPA fallback a `index.html`
- Headers de seguridad: `X-Frame-Options`, `X-Content-Type-Options`, `Referrer-Policy`

## Índices Firestore (`firestore.indexes.json`)
```json
colección: denuncias
campos: estado (ASC) + creado_en (DESC)
```
Necesario para la query: `where estado == X orderBy creado_en desc`

## Cómo desplegar manualmente
```bash
npm install -g firebase-tools
firebase login
firebase deploy --only firestore --project museo-infamia-vzla
firebase deploy --only storage --project museo-infamia-vzla
```

## GitHub Secrets requeridos
```
FIREBASE_TOKEN    # Obtener con: firebase login:ci
```