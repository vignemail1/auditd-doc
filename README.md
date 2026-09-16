# Documentation sécurité Linux

Documentation opérationnelle en français sur **auditd**, **AppArmor** et **SELinux**.

Le site est construit avec Material for MkDocs et publié automatiquement sur GitHub Pages à chaque modification de `main`.

## Développement local

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

La documentation publiée se trouve dans `docs/`. Les détails de maintenance du site sont décrits ici plutôt que dans les pages générées.
