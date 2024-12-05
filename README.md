# **Docker Registry with Proxy Cache and UI**

This project sets up a Docker Registry with **proxy cache** functionality and a **web-based UI** to manage and monitor the registry.

## **Features**

- Acts as a **proxy cache** for Docker Hub or any other upstream registry.
- Caches images locally to save bandwidth and speed up repeated pulls.
- Includes a **web UI** for easy management of cached images.
- Supports custom headers and configurations for enhanced flexibility.

---

## **Project Structure**

```
.
├── docker-compose.yml      # Docker Compose configuration file
├── data/                   # Directory to store cached images
└── config/                 # Directory for configuration files
    └── server/             # Directory for registry configuration
        └── config.yml      # Registry configuration file for proxy cache
```

---

## **Services**

### 1. **Docker Registry Server**

- **Image**: `registry:latest`
- **Ports**: `5000`
- **Features**:
  - Acts as a private Docker Registry.
  - Caches images from the upstream registry (`https://registry-1.docker.io` by default).
  - Configurable via `config.yml`.

### 2. **Docker Registry UI**

- **Image**: `joxit/docker-registry-ui:main`
- **Ports**: `1091`
- **Features**:
  - Provides a web-based interface to view and manage the Docker registry.
  - Can display image tags, catalogs, and other metadata.

---

## **Getting Started**

### 1. **Clone the Repository**

```bash
git clone <repository-url>
cd <repository-directory>
```

---

### 2. **Prepare Configuration Files**

Ensure the following files and directories exist:

#### `config/server/config.yml`

```yaml
version: 0.1
proxy:
  remoteurl: https://registry-1.docker.io
log:
  level: debug
  fields:
    service: registry
    environment: development
storage:
  cache:
    blobdescriptor: inmemory
  filesystem:
    rootdirectory: /var/lib/registry
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
health:
  storagedriver:
    enabled: true
    interval: 10s
    threshold: 3
```

#### Directory Structure

Ensure the following directory exists:

```bash
mkdir -p registry/data
```

---

### 3. **Start the Services**

Run the following command to start the registry and UI:

```bash
docker-compose up -d
```

---

### 4. **Access the Services**

- **Registry**: Accessible at `http://<your-ip>:5000`
- **Web UI**: Accessible at `http://<your-ip>:1091`

---

## **Using the Proxy Cache**

### Configure Docker Daemon

On your Docker host, configure Docker to use the registry as a mirror:

1. Edit the Docker daemon configuration file (`/etc/docker/daemon.json`):

   ```json
   {
     "registry-mirrors": ["http://<your_registry_ip>:5000"]
   }
   ```

   Replace `<your_registry_ip>` with the IP or hostname of the Docker Registry server.

2. Restart Docker:
   ```bash
   sudo systemctl restart docker
   ```

### Pull Images

When you pull an image, it will first check the local cache. If unavailable, it fetches the image from Docker Hub and stores it for future use:

```bash
docker pull nginx
```

---

## **Environment Variables**

### Docker Registry UI

| Variable                 | Description                             | Default Value                        |
| ------------------------ | --------------------------------------- | ------------------------------------ |
| `SINGLE_REGISTRY`        | Enables single registry mode            | `true`                               |
| `REGISTRY_TITLE`         | Title displayed on the UI               | `Docker Registry UI`                 |
| `NGINX_PROXY_PASS_URL`   | URL of the Docker Registry              | `http://docker-registry-server:5000` |
| `DELETE_IMAGES`          | Allows deletion of images               | `false`                              |
| `SHOW_CONTENT_DIGEST`    | Displays image content digest           | `true`                               |
| `TAGLIST_PAGE_SIZE`      | Number of tags displayed per page       | `100`                                |
| `CATALOG_ELEMENTS_LIMIT` | Limit on the number of catalog elements | `1000`                               |

---

## **Customizing the Registry**

You can modify `registry/config.yml` to:

- Change the upstream registry URL (`proxy.remoteurl`).
- Enable TLS by adding a `tls` section.
- Configure storage options.

For example, to use TLS:

```yaml
http:
  addr: :5000
  headers:
    X-Content-Type-Options: [nosniff]
  tls:
    certificate: /path/to/cert.pem
    key: /path/to/key.pem
```

---

## **Stopping the Services**

To stop the running containers:

```bash
docker-compose down
```

---

## **Troubleshooting**

1. **Verify Logs**
   Check the logs of the registry or UI services for errors:

   ```bash
   docker logs docker-registry-server
   docker logs docker-registry-ui
   ```

2. **Check Configuration Files**
   Ensure `config.yml` is properly configured and paths are correct.

3. **Verify Network Connectivity**
   Make sure your Docker host can reach the upstream registry (e.g., Docker Hub).

---

## **Future Enhancements**

- Add authentication for secure access to the registry.
- Enable HTTPS for better security.
- Monitor usage metrics with tools like Prometheus and Grafana.
