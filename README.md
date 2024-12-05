# **Docker Registry with Proxy Cache and UI**

This project sets up a Docker Registry with **proxy cache** functionality and a **web-based UI** to manage and monitor the registry.

---

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

### 3. **Start the Services**

Run the following command to start the registry and UI:

```bash
docker-compose up -d
```

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

---

## **Using the Proxy Cache to Pull Images**

When using this proxy cache setup, Docker Hub images must be prefixed with `library/` for official images. Additionally, to pull images explicitly from your private registry proxy cache, you must specify the registry host and port in the image reference.

### **Pulling Images via the Proxy Cache**

To pull images from Docker Hub and cache them through the registry proxy, use the following format:

```bash
docker pull <registry_host>:<registry_port>/library/<image_name>:<tag>
```

#### **Examples:**

1. **Pulling the Latest Alpine Image**:

   ```bash
   docker pull <registry_host>:<registry_port>/library/alpine:latest
   ```

2. **Pulling a Specific Tag of Nginx**:

   ```bash
   docker pull <registry_host>:<registry_port>/library/nginx:1.25
   ```

3. **Pulling and Using Ubuntu**:
   ```bash
   docker pull <registry_host>:<registry_port>/library/ubuntu:22.04
   docker run -it <registry_host>:<registry_port>/library/ubuntu:22.04
   ```

Replace `<registry_host>` with the IP or hostname of your registry, and `<registry_port>` with the port exposed for the registry (default: `5000`).

### **Why Use `library/` Prefix?**

Official images on Docker Hub reside under the `library` namespace. Without the prefix, your proxy cache will not correctly locate the images.

#### **Incorrect Example**:

```bash
docker pull <registry_host>:<registry_port>/alpine:latest
```

This will fail because `alpine:latest` is in the `library/` namespace.

### **Direct Pulling Without Proxy Cache**

If you need to bypass the proxy and pull directly from Docker Hub, simply omit the registry host and port:

```bash
docker pull alpine:latest
```

However, using the proxy cache allows caching and faster retrieval of images in the future.

### **Checking Cached Images**

After pulling images via the proxy, they are stored in the local registry cache. You can view these images through the **Docker Registry UI**:

1. Open the UI at `http://<your_registry_host>:1091`.
2. Browse the list of cached images under the **Catalog** section.

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
