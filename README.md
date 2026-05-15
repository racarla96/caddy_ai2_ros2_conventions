# Caddy AI2 ROS2 Conventions

## Objective

This document defines the repository organization and development workflow used for ROS 2 projects.

The main idea is to clearly separate:

- Stable and reusable production code
- Development and testing utilities
- Experimental and feature development work

This approach helps achieve:

- Clean and deployable stable branches
- Reduced contamination from experimental tooling
- Easier continuous integration
- Better maintainability
- Improved collaboration in robotics teams

---

# Philosophy

The workflow is based on a model composed of:

- Stable branch
- Development/Staging branch
- Feature branches
- Progressive promotion workflow

Code evolves progressively from experimentation to validated production-ready software.

---

# Branch Structure

## Stable branch

```text
jazzy
````

This branch contains only the minimum required components to use the package.

### Must include

* Stable and validated code
* ROS 2 nodes
* Minimal required configuration
* Essential launch files
* Public interfaces
* Strictly necessary dependencies

### Must not include

* ROS bag files
* Temporary scripts
* Benchmarking tools
* Experimental configuration
* Debug visualization
* Heavy simulation assets not required for deployment

### Purpose

This branch should always remain:

* Reproducible
* Clean
* Portable
* Deployable on real robots
* Easy to reuse by other packages

---

## Development branch

```text
jazzy-dev
```

This is the active integration and development branch.

All new functionality is validated here before being promoted to stable.

### May include

* Testing launch files
* Extended configuration
* Debugging utilities
* Helper scripts
* Benchmarking tools
* ROS bags
* RViz visualization
* Experimental configuration
* Simulation assets
* Calibration tools

### Purpose

Acts as:

* Integration branch
* Staging branch
* Controlled sandbox

This is the primary R&D environment.

---

## Feature branches

```text
feat/new-feature
feat/navigation-refactor
feat/new-controller
```

New functionality is developed in isolated branches.

### Rules

* Always created from `jazzy-dev`
* Integrated first into `jazzy-dev`
* Promoted to `jazzy` only after validation

---

# Workflow

## 1. Create a new feature branch

```bash
git checkout jazzy-dev
git pull

git checkout -b feat/new-feature
```

---

## 2. Development phase

This phase allows:

* Experimentation
* Testing
* Refactoring
* Rapid prototyping

---

## 3. Integrate into development

Once validated:

```bash
git checkout jazzy-dev
git merge feat/new-feature
```

At this stage:

* Integration tests
* Simulation
* Functional validation
* Benchmarking

are performed.

---

## 4. Promote to stable

When functionality is considered stable:

```bash
git checkout jazzy
git merge jazzy-dev
```

This promotion process should remain controlled and intentional.

---

# Promotion Philosophy

The complete workflow is:

```text
feat/* -> jazzy-dev -> jazzy
```

This implements:

* A promotion workflow
* An integration-first workflow

Not everything present in `jazzy-dev` should automatically reach `jazzy`.

---

# Recommended Repository Organization

## Configuration

Separate development and production configuration:

```text
config/
├── dev/
└── prod/
```

---

## Launch files

```text
launch/
├── dev/
└── prod/
```

---

## Scripts

```text
scripts/
├── debug/
├── calibration/
└── tools/
```

---

# Advantages

## 1. Clean stable branch

End users receive only what is strictly required.

---

## 2. Better maintainability

Avoids mixing:

* production
* simulation
* debugging
* experimentation

inside the same deployment branch.

---

## 3. Scalability

This model scales well for projects involving:

* multiple robots
* simulation
* real hardware
* CI/CD
* perception
* navigation
* control systems

---

## 4. Safer integration

New functionality passes through an intermediate validation stage before reaching stable deployment.

---

# Recommended Naming Conventions

## Features

```text
feat/feature-name
```

## Fixes

```text
fix/fix-name
```

## Experiments

```text
experiment/name
```

## Refactors

```text
refactor/name
```

---

# Additional Recommendations

## Keep `jazzy` minimal

The cleaner the stable branch remains:

* the easier deployment becomes
* the easier reuse becomes
* the fewer unnecessary dependencies are introduced

---

## Use ROS 2 overlays

This strategy works especially well when combined with overlay workspaces:

```text
underlay -> stable
overlay  -> experimental
```

---

# Summary

This repository uses a branching strategy based on:

* Stable branches
* Development branches
* Feature branches
* Progressive promotion workflows

to maintain:

* stability
* maintainability
* modularity
* experimentation capability

without compromising production-ready software quality.

## NOTA:

Por defecto, la documentación de los READMEs será en inglés.

## Objetivo

Este documento define la estrategia de organización y desarrollo utilizada en los repositorios de proyectos ROS 2.

La idea principal es separar claramente:

- Código estable y reutilizable
- Herramientas de desarrollo y testing
- Desarrollo de nuevas funcionalidades

De esta forma se consigue:

- Mantener una rama limpia y desplegable
- Evitar contaminación de herramientas experimentales
- Facilitar integración continua
- Simplificar validación y mantenimiento
- Mejorar colaboración en equipos de robótica

---

# Filosofía

El flujo de trabajo se basa en un modelo de:

- Stable branch
- Development/Staging branch
- Feature branches
- Progressive promotion workflow

Esto significa que el código evoluciona progresivamente desde desarrollo experimental hasta código estable.

---

# Estructura de ramas

## Rama estable

```text
jazzy
````

Contiene únicamente lo mínimo imprescindible para utilizar el paquete.

### Debe incluir

* Código funcional y validado
* Nodos ROS 2 estables
* Configuración mínima necesaria
* Launch files esenciales
* Interfaces públicas
* Dependencias estrictamente necesarias

### No debe incluir

* Bags de prueba
* Scripts temporales
* Herramientas de benchmarking
* Configuración experimental
* Visualización de debugging
* Assets pesados de simulación innecesarios

### Objetivo

Esta rama debe ser:

* Reproducible
* Limpia
* Portable
* Desplegable en robots reales
* Fácil de reutilizar por otros paquetes

---

## Rama de desarrollo

```text
jazzy-dev
```

Es la rama de integración y desarrollo activo.

Aquí se valida el trabajo antes de promocionarlo a estable.

### Puede incluir

* Launch files de testing
* Configuración extendida
* Herramientas de debugging
* Scripts auxiliares
* Benchmarking
* Bags ROS
* Visualización RViz
* Configuración experimental
* Simulación
* Herramientas de calibración

### Objetivo

Actúa como:

* Integration branch
* Staging branch
* Sandbox controlado

Es el entorno principal de I+D.

---

## Ramas feature

```text
feat/nueva-feature
feat/navigation-refactor
feat/new-controller
```

Las nuevas funcionalidades se desarrollan en ramas independientes.

### Reglas

* Siempre nacen desde `jazzy-dev`
* Se integran primero en `jazzy-dev`
* Solo pasan a `jazzy` cuando están validadas

---

# Flujo de trabajo

## 1. Crear nueva feature

```bash
git checkout jazzy-dev
git pull

git checkout -b feat/nueva-feature
```

---

## 2. Desarrollar

Durante esta fase se permite:

* Experimentación
* Testing
* Refactors
* Pruebas rápidas

---

## 3. Integrar en desarrollo

Una vez validada:

```bash
git checkout jazzy-dev
git merge feat/nueva-feature
```

Aquí se realizan:

* Tests de integración
* Simulación
* Validación funcional
* Benchmarks

---

## 4. Promoción a estable

Cuando la funcionalidad es suficientemente estable:

```bash
git checkout jazzy
git merge jazzy-dev
```

Esta promoción debe ser controlada.

---

# Filosofía de promoción

El flujo completo es:

```text
feat/* -> jazzy-dev -> jazzy
```

Esto implementa un:

* Promotion workflow
* Integration-first workflow

No todo lo que existe en `jazzy-dev` debe promocionarse automáticamente a `jazzy`.

---

# Organización recomendada

## Configuración

Separar configuración de desarrollo y producción:

```text
config/
├── dev/
└── prod/
```

---

## Launch files

```text
launch/
├── dev/
└── prod/
```

---

## Scripts

```text
scripts/
├── debug/
├── calibration/
└── tools/
```

---

# Ventajas

## 1. Rama estable limpia

El usuario final obtiene únicamente lo necesario.

---

## 2. Mejor mantenibilidad

Se evita mezclar:

* producción
* simulación
* debugging
* experimentación

---

## 3. Escalabilidad

Este modelo escala bien en proyectos con:

* múltiples robots
* simulación
* hardware real
* CI/CD
* percepción
* navegación
* control

---

## 4. Integración segura

Las nuevas funcionalidades pasan por una fase intermedia antes de llegar a estable.

---

# Convenciones recomendadas

## Features

```text
feat/nombre-feature
```

## Fixes

```text
fix/nombre-fix
```

## Experimentos

```text
experiment/nombre
```

## Refactors

```text
refactor/nombre
```

---

# Recomendaciones adicionales

## Mantener `jazzy` minimalista

Cuanto más limpia sea la rama estable:

* más fácil será desplegar
* más fácil será reutilizar
* menos dependencias innecesarias existirán

---

## Usar overlays de ROS 2

Muy recomendable combinar esta estrategia con workspaces overlay:

```text
underlay -> estable
overlay  -> experimental
```

---

# Resumen

Este repositorio utiliza una estrategia de branching basada en:

* Stable branch
* Development branch
* Feature branches
* Progressive promotion workflow

Con el objetivo de mantener:

* estabilidad
* mantenibilidad
* modularidad
* capacidad de experimentación

sin comprometer el código de producción.


