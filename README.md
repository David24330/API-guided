# 📘 SkillHub EC04 — Guide API Symfony (JWT + Swagger + Tests + Curl)

**Objectif :** construire une API back-end sécurisée avec Symfony 7.4, JWT, Doctrine, SQLite, Swagger et PHPUnit.

---

## ⚠️ Important avant de commencer

Dans les contrôleurs Symfony 7+, utiliser les attributs de route :

```php
use Symfony\Component\Routing\Attribute\Route;
```

Pour un JSON propre avec le Serializer, importer :

```php
use Symfony\Component\Serializer\Attribute\Groups;
```

Dans les entités, ajouter les groupes sur les champs exposés par l'API :

```php
#[Groups(['formation:read'])]
```

---

## 1. API Formation (CRUD minimal)

### GET all formations

```php
#[Route('/api/formations', methods: ['GET'])]
public function index(FormationRepository $repo): JsonResponse
{
    return $this->json(
        $repo->findAll(),
        200,
        [],
        ['groups' => 'formation:read']
    );
}
```

### GET formation by id

```php
#[Route('/api/formations/{id}', methods: ['GET'])]
public function show(Formation $formation): JsonResponse
{
    return $this->json(
        $formation,
        200,
        [],
        ['groups' => 'formation:read']
    );
}
```

### POST formation

```php
#[Route('/api/formations', methods: ['POST'])]
public function create(Request $request, EntityManagerInterface $em): JsonResponse
{
    $data = json_decode($request->getContent(), true);

    $formation = new Formation();
    $formation->setTitle($data['title']);

    $em->persist($formation);
    $em->flush();

    return $this->json($formation, 201);
}
```

### DELETE formation

```php
#[Route('/api/formations/{id}', methods: ['DELETE'])]
public function delete(Formation $formation, EntityManagerInterface $em): JsonResponse
{
    $em->remove($formation);
    $em->flush();

    return $this->json(['message' => 'deleted']);
}
```

---

## 2. Test de l’API sans Postman (Curl)

### GET formations

```bash
curl http://127.0.0.1:8000/api/formations
```

### GET formation by id

```bash
curl http://127.0.0.1:8000/api/formations/1
```

### POST formation

```bash
curl -X POST http://127.0.0.1:8000/api/formations \
-H "Content-Type: application/json" \
-d '{"title":"Symfony API"}'
```

### DELETE formation

```bash
curl -X DELETE http://127.0.0.1:8000/api/formations/1
```

---

## 3. JWT Auth (obligatoire examen)

### Suivre le cours d'API slide 136


### Installation

```bash
composer require lexik/jwt-authentication-bundle
```

### Générer les clés

```bash
php bin/console lexik:jwt:generate-keypair
```

### Configuration `.env`

```env
JWT_SECRET_KEY=%kernel.project_dir%/config/jwt/private.pem
JWT_PUBLIC_KEY=%kernel.project_dir%/config/jwt/public.pem
JWT_PASSPHRASE=
```

### Login API

**Route :** `POST /api/login`

**Body :**

```json
{
  "email": "test@test.com",
  "password": "123456"
}
```

### Réponse attendue

```json
{
  "token": "eyJ0eXAiOiJKV1QiLCJhbGciOi..."
}
```

### Curl login JWT

```bash
curl -X POST http://127.0.0.1:8000/api/login \
-H "Content-Type: application/json" \
-d '{"email":"test@test.com","password":"123456"}'
```

### Curl avec token

```bash
curl http://127.0.0.1:8000/api/formations \
-H "Authorization: Bearer TON_TOKEN"
```

---

## 4. Swagger (Nelmio API Doc)

### Installation

```bash
composer require nelmio/api-doc-bundle
```

### Configuration `config/packages/nelmio_api_doc.yaml`

```yaml
nelmio_api_doc:
    documentation:
        info:
            title: SkillHub API
            version: 1.0.0
    areas:
        default:
            path_patterns:
                - ^/api
```

### URL Swagger

```text
http://127.0.0.1:8000/api/doc
```

---

## 5. Tests PHPUnit (important examen)

### Générer un test

```bash
php bin/console make:test
```

### Lancer les tests

```bash
php bin/phpunit
```

### Test GET formations

```php
public function testGetFormations(): void
{
    $client = static::createClient();

    $client->request('GET', '/api/formations');

    $this->assertResponseIsSuccessful();
}
```

**Explication :**
- crée un client HTTP simulé ;
- appelle l’API ;
- vérifie que le status est 200.

### Test POST inscription non autorisé sans JWT

```php
public function testUnauthorizedPostInscription(): void
{
    $client = static::createClient();

    $client->request('POST', '/api/inscriptions', [], [], [], json_encode([
        'userName' => 'test',
        'email' => 'test@test.com',
        'formation' => 1
    ]));

    $this->assertResponseStatusCodeSame(401);
}
```

### Ce que doivent couvrir les tests

En examen, les tests doivent vérifier :
- GET OK
- POST OK ou KO
- sécurité JWT (401 sans token)
- endpoints principaux fonctionnels

---

## 6. Checklist examen EC04

- API CRUD Formation
- JWT login fonctionnel
- routes protégées
- Swagger filtré (`/api` uniquement)
- tests PHPUnit OK
- Curl fonctionnel sans Postman
- JSON propre avec Groups

---

## 7. Résumé fonctionnel

L’API doit permettre :
- authentification JWT ;
- CRUD Formation ;
- JSON propre avec `Groups` ;
- sécurisation des endpoints ;
- documentation Swagger ;
- tests automatisés ;
- tests manuels via Curl.
