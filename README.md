# VTEX Health Check

## Que es esto?

Repo generico para mantener vivos servicios VTEX IO mediante pings periodicos.

VTEX IO apaga los workers de un servicio despues de un periodo de inactividad (TTL). Cuando el servicio se apaga, la siguiente request sufre un **cold start** que puede tardar varios segundos. Este repo contiene un **GitHub Actions cron** que hace ping a los servicios para evitar que VTEX los mate por inactividad.

## Como funciona?

1. GitHub Actions ejecuta un workflow cada 45 minutos
2. El workflow hace `GET` a cada URL configurada en la variable `ENDPOINTS`
3. El servicio debe responder `200 OK` (idealmente con un middleware que detecte `?status=true` y responda sin ejecutar logica de negocio)
4. Si alguna respuesta no es 200, el job falla y GitHub envia notificacion

## Como agregar un nuevo servicio?

Editar `.github/workflows/health-check.yml` y agregar la URL en la variable `ENDPOINTS`:

```yaml
env:
  ENDPOINTS: |
    https://healcheck--naldoqa.myvtex.com/_v/loan-list?status=true
    https://workspace--account.myvtex.com/_v/mi-otra-ruta?status=true
    https://workspace--account.myvtex.com/_v/otro-servicio?status=true
```

Una URL por linea. El workflow hace ping a todas.

## Requisito en el servicio VTEX IO

Cada servicio debe tener un middleware que intercepte `?status=true` y responda 200 sin ejecutar logica de negocio:

```typescript
// node/middlewares/statusCheck.ts
export async function statusCheck(ctx: Context, next: () => Promise<void>) {
  if (ctx.query.status === 'true') {
    ctx.status = 200
    ctx.body = { status: 'ok', timestamp: new Date().toISOString() }
    return
  }
  await next()
}
```

Registrar como primer middleware en todas las rutas del `index.ts`.

## Configuracion

| Parametro | Valor | Como cambiar |
|-----------|-------|--------------|
| Frecuencia | Cada 45 min | Modificar cron en `schedule` |
| Endpoints | Ver `ENDPOINTS` en workflow | Agregar/quitar URLs |
| Timeout | 30s por request | Modificar `--max-time` en curl |

## Ejecucion manual

Ir a **Actions > VTEX Health Check Ping > Run workflow** en GitHub para disparar un ping manual.

## Servicios monitoreados

| Servicio | Account | Workspace | URL |
|----------|---------|-----------|-----|
| _Agregar servicios aqui_ | — | — | — |
