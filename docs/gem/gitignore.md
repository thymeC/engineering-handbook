# gitignore

gitignore is a file that tells git which files to ignore.

**knowledge point1**: I usually add some temp file for development, well it always spoil my git commit. Well, from this gitignore file, I know that I can put those temp files in the ignore list like temp/ or tmp/.

**knowledge point2**: add category to the gitignore file is a good idea.

This .gitignore file:
- Python-specific:
    - Ignores compiled Python files
    - Ignores distribution/packaging files
    - Ignores Python virtual environments
- IDE-specific:
    - VSCode settings and workspaces
    - PyCharm project files
    - Jupyter notebook checkpoints
- Development:
    - Environment files
    - Log files
    - Test coverage reports
    - Type checking cache
    - Test cache
- System:
    - macOS system files
    - Windows system files
    - Temporary files
- Project-specific:
    - Temporary directories
    - Local development settings

```
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# Virtual Environment
venv/
env/
ENV/
.env
.venv
env.bak/
venv.bak/

# IDE - VSCode
.vscode/
*.code-workspace
.history/

# IDE - PyCharm
.idea/
*.iml
*.iws
*.ipr
.idea_modules/

# IDE - Jupyter Notebook
.ipynb_checkpoints
*.ipynb

# Logs
*.log
logs/
log/

# Local development settings
.env.local
.env.development.local
.env.test.local
.env.production.local

# Coverage reports
htmlcov/
.tox/
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
.hypothesis/

# mypy
.mypy_cache/
.dmypy.json
dmypy.json

# pytest
.pytest_cache/

# System Files
.DS_Store
Thumbs.db
*.swp
*.swo

# Project specific
tmp/
temp/
```