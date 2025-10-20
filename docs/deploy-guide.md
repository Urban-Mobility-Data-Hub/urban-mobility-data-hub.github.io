---
hide:
  - toc
  - navigation
---

# Guía de Despliegue del Dataspace

Esta guía explica cómo desplegar el entorno del **Dataspace** paso a paso, tanto en local como en un entorno de pruebas o producción.

---

## Requisitos del sistema

Dado que el despliegue local instala **2 instancias de conector** y un **trust-anchor**, se recomienda utilizar una máquina lo suficientemente potente.  
Aunque **16 GB de RAM** podrían ser suficientes, se aconseja disponer de **más de 24 GB**.  
El despliegue se ha construido y probado en **Ubuntu**, aunque la mayoría de las demás distribuciones de Linux también deberían funcionar.

---

## Dependencias necesarias

El despliegue local intenta estar lo más **desacoplado posible del sistema anfitrión** para reducir los requisitos, pero aun así necesita los siguientes programas:

- **Maven**  
- **Java Development Kit (JDK)** — al menos **versión 17**  
- **Docker**

---

## Herramientas recomendadas

Para interactuar con el sistema, también son útiles las siguientes herramientas:

- **kubectl**  
- **curl**  
- **jq**  
- **yq**

---

## Despliegue local

Clonar el repositorio:

```bash
   git clone https://github.com/urban-mobility-data-hub/local-dataspace.git
   cd local-dataspace
```

Desplegar el dataspace:

```bash
    mvn clean deploy -Plocal
```

Añadir el contexto a KUBECONFIG:

```bash
    export  KUBECONFIG=$(pwd)/target/k3s.yaml
```

Ver despliegue en tiempo real:

```bash
    watch kubectl get all --all-namespaces
```


