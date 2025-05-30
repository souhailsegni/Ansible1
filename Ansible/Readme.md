# Ansible
Let's play Some Ansible PLayBooks

# Ansible Minikube Flask & Postgres Deployment Guide

## Prerequisites

- [Minikube](https://minikube.sigs.k8s.io/docs/start/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Ansible](https://docs.ansible.com/)
- Python 3 with the `kubernetes` library (`python3-kubernetes` package or use a virtualenv)
- Docker images for Flask (`monitoring-web:latest`) and Postgres (`postgres:latest`) built locally

---

## 1. Clone or Prepare Your Project Directory

```sh
cd ~/Desktop/Ansible/Ansible
```

---

## 2. Create `inventory.ini`

```ini
[local]
localhost ansible_connection=local
```

---

## 3. Prepare Kubernetes Manifests

Place your `postgres.yaml` and `flask.yaml` manifests in a `k8s/` directory:

```sh
mkdir -p k8s
mv postgres.yaml k8s/
mv flask.yaml k8s/
```

---

## 4. Create or Update `playbook.yaml`

Make sure your playbook uses the correct paths:

```yaml
- name: Deploy existing Flask & Postgres images on Minikube
  hosts: local
  gather_facts: no
  vars:
    namespace: flask-app
    flask_image: app:latest
    postgres_image: postgres:latest

  tasks:
    - name: Start Minikube if needed
      command: minikube start
      register: mk
      failed_when: mk.rc not in [0,104]

    - name: Load local Flask image into Minikube
      command: minikube image load {{ flask_image }}

    - name: Load local Postgres image into Minikube
      command: minikube image load {{ postgres_image }}

    - name: Create namespace
      kubernetes.core.k8s:
        state: present
        definition:
          apiVersion: v1
          kind: Namespace
          metadata:
            name: "{{ namespace }}"

    - name: Apply Postgres stack
      kubernetes.core.k8s:
        state: present
        namespace: "{{ namespace }}"
        src: k8s/postgres.yaml

    - name: Apply Flask app stack
      kubernetes.core.k8s:
        state: present
        namespace: "{{ namespace }}"
        src: k8s/flask.yaml

    - name: Wait for Postgres pod to be Ready
      kubernetes.core.k8s_info:
        kind: Pod
        namespace: "{{ namespace }}"
        label_selectors:
          - app=postgres
      register: pg_pod
      until: pg_pod.resources | length > 0 and pg_pod.resources[0].status.phase == "Running"
      retries: 12
      delay: 5

    - name: Wait for Flask pod to be Ready
      kubernetes.core.k8s_info:
        kind: Pod
        namespace: "{{ namespace }}"
        label_selectors:
          - app=flask
      register: flask_pod
      until: flask_pod.resources | length > 0 and flask_pod.resources[0].status.phase == "Running"
      retries: 12
      delay: 5

    - name: Show access info
      debug:
        msg: |
          Your app should now be up!
          • Flask → http://$(minikube ip):30007
          • Postgres is available internally at “postgres:5432”
```

> **Note:**  
> Replace `community.kubernetes.k8s` with `kubernetes.core.k8s` as shown above to avoid deprecation warnings.

---

## 5. Install Python Kubernetes Client

**On Ubuntu/Debian:**
```sh
sudo apt install python3-kubernetes
```
**Or, using a virtual environment:**
```sh
python3 -m venv venv
source venv/bin/activate
pip install kubernetes
```

---

## 6. Run the Playbook

```sh
ansible-playbook -i inventory.ini playbook.yaml
```

---

## 7. Access Your Services

Get your Minikube IP:
```sh
minikube ip
```

- **Flask:**  
  Visit `http://<minikube-ip>:30007`
- **Postgres:**  
  Accessible internally at `postgres:5432` within the cluster.

---

## Troubleshooting

- If pods are stuck in `Pending`, check PVC/PV status:
  ```sh
  kubectl get pvc -n flask-app
  kubectl get pv
  ```
- If you see image pull errors, ensure you loaded your local images into Minikube:
  ```sh
  minikube image load monitoring-web:latest
  minikube image load postgres:latest
  ```
- For more details, describe pods:
  ```sh
  kubectl describe pod <pod-name> -n flask-app
  ```

---

## Notes

- Avoid using reserved variable names like `namespace` in Ansible.
- Update deprecated module names as shown above.
- Ensure your YAML manifest paths match your playbook.

---

**Happy automating!**
