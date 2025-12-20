# 🚀 Guia de Instalação Alfresco - Produção com Nginx Proxy Manager

## 📋 Pré-requisitos

- VPS Ubuntu com Docker e Docker Compose instalados
- Domínio `dms.eris.cv` apontando para o IP do VPS
- Nginx Proxy Manager já instalado e a funcionar
- Mínimo 6GB RAM disponível
- Portas 80 e 443 abertas

---

## 🚀 Instalação

### 1. Clonar Repositório

```bash
cd /opt
git clone https://github.com/rubenpires333/alfresco.git alfresco
cd alfresco/docker-compose
```

### 2. Iniciar Alfresco

```bash
docker compose -f production-compose-nginxpm.yaml up -d
```

### 3. Aguardar Inicialização

Aguardar 3-5 minutos para todos os serviços iniciarem:

```bash
# Ver logs
docker compose -f production-compose-nginxpm.yaml logs -f

# Verificar status
docker compose -f production-compose-nginxpm.yaml ps
```

---

## 🔧 Configuração Nginx Proxy Manager

### 1. Conectar NPM à Rede do Alfresco

```bash
docker network connect docker-compose_alfresco-network nginx-proxy-manager
docker restart nginx-proxy-manager
```

### 2. Configurar Proxy Host no NPM

1. Aceder a: `http://SEU_IP:81`
2. **Proxy Hosts** → **Add Proxy Host** (ou editar existente)

#### Details Tab:
- **Domain Names**: `dms.eris.cv`
- **Scheme**: `http`
- **Forward Hostname/IP**: `localhost`
- **Forward Port**: `8080`
- ✅ **Websockets Support**

#### SSL Tab:
- **SSL Certificate**: Request a new SSL Certificate
- ✅ **Force SSL**
- ✅ **HTTP/2 Support**

#### Custom Locations Tab:
Colar esta configuração:

```nginx
location /alfresco {
    proxy_pass http://docker-compose-alfresco-1:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Port $server_port;
    proxy_read_timeout 300s;
    proxy_connect_timeout 75s;
    client_max_body_size 0;
}

location /share {
    proxy_pass http://docker-compose-share-1:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Port $server_port;
    proxy_read_timeout 300s;
    proxy_connect_timeout 75s;
    client_max_body_size 0;
}

location = / {
    return 301 /share/;
}
```

3. **Salvar**

---

## ✅ Verificação

```bash
# Verificar containers
docker compose -f production-compose-nginxpm.yaml ps

# Testar acesso
curl -I https://dms.eris.cv/share/ -k
```

Aceder no browser: **https://dms.eris.cv/share/**

**Login padrão:**
- Utilizador: `admin`
- Password: `admin`

---

## 🔧 Comandos Úteis

```bash
# Ver logs
docker compose -f production-compose-nginxpm.yaml logs -f

# Reiniciar serviços
docker compose -f production-compose-nginxpm.yaml restart

# Parar serviços
docker compose -f production-compose-nginxpm.yaml down

# Ver status
docker compose -f production-compose-nginxpm.yaml ps
```

---

## 📝 Notas Importantes

1. **Nomes dos Containers**: Usar `docker-compose-alfresco-1` e `docker-compose-share-1` na configuração do NPM (não `localhost`)
2. **Rede Docker**: NPM deve estar na mesma rede que os containers do Alfresco
3. **Portas**: Share usa porta 8080 dentro do container (mapeada para 8081 no host)
4. **SSL**: Certificado Let's Encrypt é gerado automaticamente pelo NPM

---

**Instalação concluída! 🎉**

