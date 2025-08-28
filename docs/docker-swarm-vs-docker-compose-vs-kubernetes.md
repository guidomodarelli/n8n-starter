¿Qué es Docker Swarm?

1. ¿Es un orquestador "integrado" en Docker, similar a Kubernetes?
2. ¿Qué relación tiene (o no) con Docker Compose?
3. ¿Cómo se compara con Kubernetes?

---

Resumen rápido:

* **Docker Swarm** (“swarm mode”) es el **orquestador** integrado en Docker Engine para correr servicios en **múltiples
  hosts** con clustering, scheduling, escalado y *rolling updates*. ([Docker Documentation][1])
* **Kubernetes** también es un orquestador, pero con un **ecosistema y alcance mayores**; suele elegirse para
  despliegues más complejos/masivos. Swarm es más simple y está integrado a Docker; K8s es más potente y estándar de
  facto. ([Portworx][2])
* **Docker Compose** **no es** un orquestador: es una **forma de describir** una app multi-contenedor (usualmente en
  **un solo host**) y levantarla con `docker compose up`. Distinto rol. ([Docker Documentation][3])

# ¿Cómo se relacionan (Compose ↔ Swarm ↔ Kubernetes)?

* Puedes usar **un archivo Compose** para desplegar en **Swarm** con `docker stack deploy` (usa la **sintaxis “legacy”
  v3** y **no** toda la especificación Compose moderna). Por eso, claves como `deploy:` se **ignoran** en `docker
  compose up` (local), pero **sí** aplican en `docker stack deploy` (Swarm). ([Docker Documentation][4], [Stack
  Overflow][5])
* “Compose on Kubernetes” (usar `docker stack ... --orchestrator=kubernetes`) **está discontinuado/archivado**. Si
  quieres pasar de Compose a K8s hoy, usa **Kompose** para convertir a *manifests* de Kubernetes. ([GitHub][6],
  [Kubernetes][7])

# ¿Se pueden comparar Swarm y Kubernetes?

Sí: ambos orquestan contenedores (servicios, escalado, redes, secretos, *rolling updates*), pero **K8s** ofrece más
extensibilidad, *operators*, ecosistema y nube gestionada; **Swarm** prioriza **simplicidad** y la integración con
Docker. Además, **Swarm sigue con soporte empresarial** (Mirantis lo extendió hasta **2030**). ([Microsoft Learn][8],
[Portworx][2], [Mirantis][9])

# ¿Qué usar?

* **Un solo VPS / desarrollo local:** **Docker Compose**.
* **Varios nodos, alta disponibilidad simple y rápida:** **Swarm**.
* **Plataformas grandes, multi-equipo, nube gestionada y extensiones (Ingress, CSI, HPA, etc.):** **Kubernetes**.
  ([Docker Community Forums][10], [Portworx][2])

# Comandos útiles

* ¿Tu Docker está en Swarm?

  ```bash
  docker info | grep -i swarm      # “Swarm: active” o “inactive”
  ```
* Iniciar Swarm en el nodo actual:

  ```bash
  docker swarm init
  ```
* Desplegar un stack en Swarm desde un compose v3 (ojo: imágenes preconstruidas y `deploy:` aplicado):

  ```bash
  docker stack deploy -c docker-compose.yml miapp
  ```

  (Usa el formato v3 “legacy”; no toda la spec actual de Compose es compatible con `stack deploy`). ([Docker Documentation][4])

[1]: https://docs.docker.com/engine/swarm/?utm_source=chatgpt.com "Swarm mode"
[2]: https://portworx.com/knowledge-hub/kubernetes-vs-docker/?utm_source=chatgpt.com "Kubernetes vs Docker: Differences & Definitions"
[3]: https://docs.docker.com/reference/compose-file/deploy/?utm_source=chatgpt.com "Compose Deploy Specification"
[4]: https://docs.docker.com/engine/swarm/stack-deploy/?utm_source=chatgpt.com "Deploy a stack to a swarm"
[5]: https://stackoverflow.com/questions/61359375/how-to-switch-to-docker-compose-file-v3-for-applications-running-exclusively-on?utm_source=chatgpt.com "How to switch to docker Compose file v3 for applications ..."
[6]: https://github.com/docker/compose-on-kubernetes?utm_source=chatgpt.com "docker/compose-on-kubernetes: Deploy applications ..."
[7]: https://kubernetes.io/docs/tasks/configure-pod-container/translate-compose-kubernetes/?utm_source=chatgpt.com "Translate a Docker Compose File to Kubernetes Resources"
[8]: https://learn.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/swarm-mode?utm_source=chatgpt.com "Get started with swarm mode"
[9]: https://www.mirantis.com/blog/mirantis-guarantees-long-term-support-for-swarm/?utm_source=chatgpt.com "Mirantis Commits to Long-Term Support for Swarm"
[10]: https://forums.docker.com/t/single-host-deployment-docker-compose-vs-docker-stack-deploy-bridge-driver-swarm-scope-vs-docker-stack-deploy-overlay-driver/145176?utm_source=chatgpt.com "docker compose vs docker stack deploy (bridge driver ..."
