# UrbanCart

Application e-commerce construite en architecture microservices et conteneurisée avec Docker.

## Stack technique

- **Frontend** — Vue 3, Vite, Nginx (prod)
- **Auth service** — Node.js, Express, JWT
- **Product service** — Node.js, Express
- **Order service** — Node.js, Express
- **Base de données** — MongoDB
- **Reverse proxy** — Nginx
- **Orchestration** — Docker Compose

---

## Lancement du projet

### Prérequis

- Docker Desktop installé et lancé
- Le fichier `.env` présent à la racine (voir section Variables d'environnement)

### Développement
```bash
docker compose up -d --build
```

Accéder à l'application : [http://localhost:8080](http://localhost:8080)

### Production

Créer le fichier de secret JWT avant de lancer :
```bash
mkdir -p secrets
echo "urbancart_secret_key" > secrets/jwt_secret.txt
```

Puis lancer :
```bash
docker compose -f docker-compose.prod.yml up -d --build
```

Accéder à l'application : [http://localhost](http://localhost)

---

## Variables d'environnement

| Fichier | Rôle |
|---|---|
| `.env` | Variables partagées (Mongo, JWT, URLs services) |
| `.env.development` | Surcharge pour l'environnement de développement |
| `.env.production` | Surcharge pour l'environnement de production |
| `frontend/.env` | Variables Vite pour le frontend en dev |
| `frontend/.env.production` | Variables Vite pour le frontend en prod |

---

## Architecture
```
urbancart/
├── frontend/                  # Application Vue 3
│   ├── src/
│   ├── Dockerfile.dev
│   ├── Dockerfile.prod
│   └── nginx.conf
├── services/
│   ├── auth-service/          # Authentification (port 3001)
│   │   ├── Dockerfile.dev
│   │   └── Dockerfile.prod
│   ├── product-service/       # Produits & panier (port 3000)
│   │   ├── Dockerfile.dev
│   │   └── Dockerfile.prod
│   └── order-service/         # Commandes (port 3002)
│       ├── Dockerfile.dev
│       └── Dockerfile.prod
├── secrets/
│   └── jwt_secret.txt
├── docker-compose.yml
├── docker-compose.prod.yml
└── .env
```

Le frontend en production est servi par Nginx qui fait office de reverse proxy :

| Route | Service |
|---|---|
| `/api/auth` | auth-service:3001 |
| `/api/products` | product-service:3000 |
| `/api/cart` | product-service:3000 |
| `/api/orders` | order-service:3002 |

---

## Build manuel des images
```bash
docker build -t urbancart-frontend-prod:1 -f ./frontend/Dockerfile.prod ./frontend

docker build -t urbancart-auth-prod:1 -f ./services/auth-service/Dockerfile.prod ./services/auth-service

docker build -t urbancart-product-prod:1 -f ./services/product-service/Dockerfile.prod ./services/product-service

docker build -t urbancart-order-prod:1 -f ./services/order-service/Dockerfile.prod ./services/order-service
```

---

## Tester les services

### En développement

**Auth**
```bash
curl -X POST http://localhost:3001/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "password123"}'

curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "user@example.com", "password": "password123"}'
```

**Produits**
```bash
curl -X GET http://localhost:3000/api/products

curl -X POST http://localhost:3000/api/cart/add \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -d '{"userId": "<user_id>", "productId": "<product_id>", "quantity": 1}'
```

**Commandes**
```bash
curl -X POST http://localhost:3002/api/orders \
  -H "Authorization: Bearer <JWT_TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{
    "products": [{"productId": "<product_id>", "quantity": 1}],
    "shippingAddress": {"street": "12 rue de la Paix", "city": "Paris", "postalCode": "75001"}
  }'

curl -X GET http://localhost:3002/api/orders \
  -H "Authorization: Bearer <JWT_TOKEN>"
```

### En production
```bash
docker exec -it urbancart-frontend sh
```

Utiliser les noms de services Docker (`auth-service:3001`, `product-service:3000`, `order-service:3002`) à la place de `localhost`.

---

## Bonnes pratiques appliquées

**Sécurité**
- Les conteneurs de production tournent avec un utilisateur non-root (`app`)
- Le secret JWT est injecté via `secrets/jwt_secret.txt`
- Les `node_modules` et fichiers sensibles sont exclus via `.dockerignore`

**Fiabilité**
- Chaque service dispose d'un `healthcheck`
- Les services ne démarrent qu'une fois MongoDB opérationnel (`depends_on: condition: service_healthy`)
- Politique `restart: unless-stopped` sur tous les conteneurs

**Performance**
- Build multi-stage pour le frontend : Node.js pour compiler, Nginx pour servir
- `npm install --omit=dev` dans les images de production

**Développement**
- Nodemon pour le hot-reload en développement
- Volumes pour refléter les changements locaux sans rebuild

---

## Ajouter des produits
```bash
docker exec -it urbancart-mongo mongosh
use urbancart

db.products.insertMany([
  { name: "Hoodie UrbanCart", price: 59.99, description: "Coton épais, coupe oversize" },
  { name: "T-shirt Essential", price: 29.99, description: "100% coton biologique" },
  { name: "Cargo Pants", price: 79.99, description: "Poches multiples, coupe droite" }
])
```

---

## Conclusion

Ce projet m'a permis de mettre en pratique :

- La conteneurisation d'une application avec Docker
- L'orchestration de plusieurs services avec Docker Compose
- La gestion d'une architecture microservices
- La mise en place d'environnements distincts (développement et production)

Il illustre également l'importance de la séparation des responsabilités entre les services et l'utilisation de bonnes pratiques en termes de sécurité et de performance.
