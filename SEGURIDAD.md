# Recomendaciones de seguridad 🔒

Le pedí a un asistente de IA (Claude) que revisara el proyecto y me diera recomendaciones de seguridad.
Esta es la lista y lo que hice con cada una.

## Aplicadas en el código

| # | Riesgo | Qué cambié | Archivo |
|---|---|---|---|
| 1 | Si `TELEGRAM_CHAT_ID` quedaba vacío, **cualquier persona** que encontrara el bot podía usarlo y gastar mi saldo de Claude y mi cuota de Alpha Vantage. | Ahora el bot está **cerrado por defecto**: sin chat autorizado no atiende a nadie. | `mesa/telegram_io.py` |
| 2 | Algunos errores de conexión muestran la URL completa, y la URL de Telegram **lleva el token adentro**. Podía quedar en los registros públicos de Actions. | Nueva función `ocultar_llaves()` que cambia cualquier llave por `***` antes de imprimir un error. | `mesa/config.py`, `revisar_telegram.py`, `bot_continuo.py` |
| 3 | Mensajes muy largos pueden traer instrucciones raras para la IA (prompt injection) o gastar recursos. | Se ignoran mensajes de más de 40 caracteres (un `/analiza` nunca necesita más). | `mesa/comandos.py` |
| 4 | Las acciones de GitHub (`@v7`) podían cambiar sin aviso si alguien modifica esa versión. | Quedaron **fijas a un commit exacto** (SHA). | `.github/workflows/*.yml` |
| 5 | El token de GitHub se quedaba guardado en el runner después de descargar el código. | `persist-credentials: false` en el checkout. | `.github/workflows/*.yml` |
| 6 | Librerías o acciones con fallas de seguridad sin que me entere. | Activé **Dependabot**: revisa cada semana y propone actualizaciones. | `.github/dependabot.yml` |
| 7 | Una versión nueva mayor de una librería podía romper el bot. | Versiones con tope en `requirements.txt`. | `requirements.txt` |
| 8 | Las evidencias se guardaban 14 días. | Ahora 7 días. | `mesa-financiera.yml` |
| 9 | Con el cron cada 8 minutos, una ejecución lenta se podía encimar con la siguiente. | `timeout-minutes: 7` (ya existía `concurrency` para no correr dos a la vez). | `mesa-financiera.yml` |

Ya venían bien desde la plantilla: las llaves solo están en **GitHub Secrets** (nunca en el código), `.env` está en `.gitignore`,
el ticker se valida con una expresión regular, el workflow solo tiene permiso de lectura (`contents: read`) y la nota siempre
termina con el descargo de responsabilidad.

## Las que tengo que hacer yo (no se pueden hacer con código)

- [ ] **Rotar las llaves** (generar nuevas): token del bot en @BotFather (`/revoke`), llave de Claude y llave de Alpha Vantage,
      y actualizar los 4 Secrets. Las llaves aparecieron en una captura de pantalla.
- [ ] Poner un **límite de gasto** a la llave de Claude en la Claude Console.
- [ ] Activar **Secret scanning / Push protection** en Settings → Advanced Security (gratis en repos públicos):
      GitHub bloquea un push si detecta una llave.
- [ ] Tener activada la **verificación en dos pasos (2FA)** en GitHub.
- [ ] Antes de entregar, revisar que **ninguna captura** muestre una llave, ni completa ni parcial.
- [ ] No mandarle datos personales al bot.

Las notas que generan los agentes son material educativo, no recomendaciones de inversión.
