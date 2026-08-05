# Fork iTalk de `odoo/upgrade-util` — rama `19.0-italk`

## Para que existe

Los scripts de migracion de los modulos de ADHOC (`account_payment_pro`,
`stock_account_ux`, y los que vengan) importan la libreria de utilidades de
migracion de Odoo:

```python
from odoo.upgrade import util
```

Esa libreria la provee la plataforma **solo durante una migracion de version
mayor** (18 -> 19). En una actualizacion comun de modulos —el caso habitual en
Odoo.sh— no esta disponible, y el import aborta la carga de la base entera.

Este fork la deja disponible como dependencia Python del repo del cliente.

## Que cambia respecto de upstream

**Se eliminan los directorios de scripts de migracion**: `src/base`, `src/mail`
y `src/spreadsheet`. Se conserva unicamente la libreria (`src/util`) y su
helper de tests (`src/testing.py`).

**Por que.** Odoo descubre scripts de migracion recorriendo `odoo.upgrade.__path__`,
que incluye lo instalado por pip. Si se instala el paquete completo, Odoo
encuentra los scripts de `src/base/0.0.0/` y los ejecuta **en cualquier build
donde el modulo `base` cambie de version** (es decir, cada vez que la plataforma
actualiza el nucleo de Odoo). Entre ellos hay uno que hace:

```sql
UPDATE ir_module_module SET state = 'to upgrade' WHERE state = 'installed'
```

o sea, fuerza la re-actualizacion de **todos** los modulos instalados. Eso es
correcto dentro de una migracion de version mayor orquestada por la plataforma,
y es un efecto colateral grave en un build de rutina, produccion incluida.

Sin esos directorios, `odoo.upgrade` solo aporta la libreria: Odoo no encuentra
ningun directorio con nombre de modulo y no ejecuta nada por su cuenta.

## Como se consume

En el `requirements.txt` del repo del cliente:

```
git+https://github.com/iTalkarg/upgrade-util@19.0-italk
```

## Como se actualiza

Traer los cambios de upstream y volver a sacar los directorios de scripts:

```bash
git remote add upstream https://github.com/odoo/upgrade-util.git   # una sola vez
git fetch upstream
git rebase upstream/master        # resolver dejando borrados src/base, src/mail, src/spreadsheet
git push --force-with-lease origin 19.0-italk
```

Solo hace falta cuando un script de migracion de ADHOC necesite una utilidad
nueva. La libreria es estable; no requiere seguimiento continuo.

## Licencia

Upstream es LGPL-3.0 y este fork la mantiene sin cambios (ver `LICENSE`).
