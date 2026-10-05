# 🚀 Hello Site — деплой в Minikube

<div align="center">

![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white)
![Nginx](https://img.shields.io/badge/nginx-%23009639.svg?style=for-the-badge&logo=nginx&logoColor=white)
![HTML5](https://img.shields.io/badge/html5-%23E34F26.svg?style=for-the-badge&logo=html5&logoColor=white)

**Простой статический сайт, развёрнутый в локальном Kubernetes-кластере Minikube.**

[Требования](#-требования) • [Быстрый старт](#-быстрый-старт) • [Troubleshooting](#-troubleshooting)

</div>

---

## 📖 О проекте

Этот репозиторий демонстрирует полный цикл деплоя статического сайта в Kubernetes:

- 📄 Простая HTML-страница
- 🐳 Упаковка в Docker-образ на базе `nginx:alpine`
- ☸️ Разворачивание в Minikube через `Deployment` и `Service` (NodePort)
- 🌐 Доступ к сайту через браузер

Отлично подходит как «hello world» для знакомства с Kubernetes.

---

## 📋 Требования

Перед началом убедись, что установлены:

| Инструмент | Ссылка на установку |
|---|---|
| 🐳 Docker | https://docs.docker.com/get-docker/ |
| ☸️ Minikube | https://minikube.sigs.k8s.io/docs/start/ |
| ⚙️ kubectl | https://kubernetes.io/docs/tasks/tools/ |

Проверка версий:

```bash
docker --version
minikube version
kubectl version --client
```

---

## 📁 Структура проекта

```
.
├── index.html        # Статическая страница сайта
├── Dockerfile        # Сборка образа на базе nginx:alpine
├── deployment.yaml   # Kubernetes Deployment
├── service.yaml      # Kubernetes Service (NodePort)
└── README.md         # Этот файл
```

---

## ⚡ Быстрый старт

### 1️⃣ Клонируем репозиторий

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

### 2️⃣ Собираем Docker-образ

```bash
docker build -t hello-site:v1 .
```

### 3️⃣ Загружаем образ в Minikube

Minikube использует собственный Docker-демон, поэтому образ нужно явно загрузить внутрь кластера:

```bash
minikube image load hello-site:v1
```

Проверим, что образ виден:

```bash
minikube image list | grep hello-site
```

### 4️⃣ Запускаем кластер и деплоим

```bash
# Стартуем Minikube
minikube start

# Применяем манифесты
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

Проверяем статус:

```bash
kubectl get pods
kubectl get svc hello-site-svc
```

Под должен перейти в статус `Running`:

```bash
kubectl get pods -l app=hello-site
```

### 5️⃣ Открываем сайт в браузере

```bash
minikube service hello-site-svc
```

Команда автоматически откроет сайт по адресу вида `http://192.168.49.2:30080`.

Если браузер не открылся — получи ссылку вручную:

```bash
minikube service hello-site-svc --url
```

---

## 🧹 Очистка ресурсов

Когда закончишь — удали ресурсы:

```bash
kubectl delete -f deployment.yaml
kubectl delete -f service.yaml
minikube stop
```

Полностью удалить кластер:

```bash
minikube delete
```

---

## 🐛 Troubleshooting

<details>
<summary><b>Под в статусе <code>ImagePullBackOff</code> или <code>ErrImageNeverPull</code></b></summary>

Убедись, что образ загружен в Minikube:

```bash
minikube image list | grep hello-site
```

Если образа нет — выполни:

```bash
minikube image load hello-site:v1
```

</details>

<details>
<summary><b>Порт 30080 занят</b></summary>

Измени `nodePort` в `service.yaml` на другое значение в диапазоне `30000–32767` и примени манифест заново:

```bash
kubectl apply -f service.yaml
```

</details>

<details>
<summary><b><code>kubectl</code> не видит кластер</b></summary>

Проверь текущий контекст:

```bash
kubectl config current-context
```

Должно вывести `minikube`. Если нет — переключись:

```bash
kubectl config use-context minikube
```

</details>

<details>
<summary><b>Под долго в статусе <code>Pending</code></b></summary>

Посмотри детали:

```bash
kubectl describe pod <pod-name>
```

Чаще всего проблема в нехватке ресурсов — попробуй перезапустить Minikube с большим объёмом памяти:

```bash
minikube delete
minikube start --memory=4096
```

</details>

---

## 📚 Полезные команды

| Команда | Что делает |
|---|---|
| `kubectl get pods` | Список подов |
| `kubectl logs <pod>` | Логи пода |
| `kubectl describe pod <pod>` | Детали пода (для отладки) |
| `kubectl get svc` | Список сервисов |
| `kubectl get all` | Все ресурсы в namespace |
| `minikube dashboard` | Открыть веб-дашборд Kubernetes |
| `minikube service <svc>` | Открыть сервис в браузере |
| `minikube stop` | Остановить кластер |
| `minikube delete` | Полностью удалить кластер |

---

## 🛠️ Как это работает

```
┌─────────────────┐      ┌──────────────────┐      ┌─────────────────┐
│   index.html    │─────▶│  Docker-образ    │─────▶│  Minikube       │
│                 │      │  (nginx:alpine)  │      │  (Kubernetes)   │
└─────────────────┘      └──────────────────┘      └─────────────────┘
                                                          │
                                                          ▼
                                                   ┌─────────────────┐
                                                   │  Deployment     │
                                                   │  + Service      │
                                                   │  (NodePort)     │
                                                   └─────────────────┘
                                                          │
                                                          ▼
                                                   ┌─────────────────┐
                                                   │  🌐 Браузер     │
                                                   │  localhost:30080│
                                                   └─────────────────┘
```

1. **HTML-файл** копируется внутрь образа `nginx:alpine`
2. **Docker-образ** загружается в кластер Minikube
3. **Deployment** создаёт под с контейнером nginx
4. **Service** типа NodePort открывает доступ к поду снаружи
5. **Браузер** обращается к сайту по IP Minikube и порту `30080`

---

<div align="center">

**Если проект был полезен — поставь ⭐ репозиторию!**

Made with ❤️ for learning Kubernetes

</div>
