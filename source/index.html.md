---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - javascript--browser: javascript
  - javascript--node: nodejs

toc_footers:
  - <a href='https://certylink.com'>Certylink website</a>

includes:
  - errors

search: true
---

# Introduction

Bienvenue sur la documentation de l'API CertyLink Pro.
Cette API peut être utilisée principalement afin d'archiver les documents PDF de votre entreprise à partir d'un ERP ou d'une application tierce. Elle sert également à effectuer des recherches de documents, les télécharger, leur ajouter/retirer des tags etc...

Nous proposons ce service sous forme d'API REST au format JSON, l'authentification se fait à l'aide d'une clé API disponible dans l'espace d'administration sur [l'application web CertyLink Pro](https://app.certy.link).

# Authentification

> Afin d'authentifier votre entreprise, vous devez renseigner votre clé API dans le champ header "Authorization" pour chaque requête.

```shell
curl "https://api.certy.link/v1/ENDPOINT"
  -H Authorization:'APIKEY'
```

```javascript--browser
// Si vous souhaitez utiliser le module axios, référez vous à l'onglet nodejs,
// Nous utiliserons ici l'API fetch implémentée par les navigateurs récents

fetch("https://api.certy.link/v1/ENDPOINT", {
  headers: {
    Authorization: "APIKEY",
  },
});
```

```javascript--node
// Nous déclarons une instance avec axios afin d'éviter
// de réécrire l'url de base et votre clé api à chaque requête

// Ajoutez le module axios à votre projet avec npm : 'npm i axios'
const axios = require('axios');

const certyClient = axios.create({
  baseURL: "https://api.certy.link/v1/",
  headers: { Authorization: "APIKEY" }
});

// Utilisation
certyClient.get("ENDPOINT");

```

> Remplacez `APIKEY` par votre clé API et `ENDPOINT` par le point d'accès désiré.

CertyLink utilise une clé API afin d'autoriser l'accès à l'API. Elle est disponible pour les administrateurs d'un compte entreprise sur [l'application web CertyLink Pro](https://app.certy.link).

La clé API doit être incluse dans chaque requête au serveur décrites dans cette documentation, elle est passée dans le Header `Authorization` de la requête sous la forme :

`Authorization: APIKEY`

Vous pouvez également utiliser un préfixe optionnel par exemple :

`Authorization: key APIKEY`

<aside class="notice">
Remplacez <code>APIKEY</code> par votre clé personnelle.
</aside>

# Archiver un document PDF

## Méthode base64

```shell
curl "https://api.certy.link/v1/documents/upload/base64"
  -X POST
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/documents/upload/base64", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
  method: "POST",
  body: JSON.stringify({
    file: "BASE64PDFFILE",
    filename: "FILENAME",
    userId: "USERID",
    profileId: "PROFILEID",
    tagNames: ["TAGNAME", "TAGNAME2"],
  }),
});
```

```javascript--node
certyClient.post("documents/upload/base64", {
  file: "BASE64PDFFILE",
  filename: "FILENAME",
  userId: "USERID",
  profileId: "PROFILEID",
  tagNames: ["TAGNAME", "TAGNAME2"],
});
```

> Le serveur retournera une réponse en JSON de cette forme:

```json
{
  "success": true,
  "docId": "DOCUMENTID"
}
```

Ce point d'accès permet l'archivage d'un document pdf au format base64.
L'avantage de cette méthode est la simplicité de l'envoi du fichier.

### Requête HTTP

`POST https://api.certy.link/v1/documents/upload/base64`

### Paramètres du Body

| Paramètre | Requis | Description                                                                                                   |
| --------- | ------ | ------------------------------------------------------------------------------------------------------------- |
| file      | X      | Fichier pdf converti en base64                                                                                |
| filename  | X      | Nom du fichier souhaité                                                                                       |
| userId    | X      | Identifiant de l'utilisateur qui émet le document                                                             |
| profileId | X      | Identifiant du profil à utiliser (détermine le placement du qrCode, les droits d'accès et des tags à ajouter) |
| tagNames  |        | Tableau de noms de tags à ajouter(le tag sera créé si le nom n'existe pas)                                    |

### Réponse du serveur

| Paramètre | Type    | Description                                           |
| --------- | ------- | ----------------------------------------------------- |
| success   | Boolean | Retourne `true` si la requête est traitée avec succès |
| docId     | String  | Identifiant unique du document                        |

## Méthode multipart/form-data

Cette méthode consiste à envoyer le fichier en binaire en même temps que les données associées.
Elle est plus optimisée que la méthode en base64 pour les fichiers volumineux.

```shell
curl "https://api.certy.link/v1/documents/upload"
    -F file=@FILE.PDF
    -F userId=USERID
    -F profileId=PROFILEID
  -H Authorization:'APIKEY'
```

```javascript--browser
const formData = new FormData();
formData.append("userId", "USERID");
formData.append("profileId", "PROFILEID");
formData.append("file", "FILEINPUTBINARY");
const tagNames = ["TAGNAME1", "TAGNAME2"];
tagNames.forEach(tagName => {
  formData.append("tagNames[]", tagName);
});
fetch("https://api.certy.link/v1/documents/upload", {
  headers: {
    Authorization: "APIKEY",
  },
  method: "POST",
  body: formData,
});
```

```javascript--node
const FormData = require('form-data') // module à installer pour utiliser formData côté serveur
const fs = require('fs');

const formData = new FormData();
formData.append("userId", "USERID");
formData.append("profileId", "PROFILEID");
formData.append("file", fs.createReadStream('FILEPATH'), 'FILENAME'); // Envoie le fichier en stream
const tagNames = ["TAGNAME1", "TAGNAME2"];
tagNames.forEach(tagName => {
  formData.append("tagNames[]", tagName);
});
certyClient.post("documents/upload", formData, {
  headers: formData.getHeaders()
});
```

> Le serveur retournera une réponse en JSON de cette forme:

```json
{
  "success": true,
  "docId": "DOCUMENTID"
}
```

### Requête HTTP

`POST https://api.certy.link/v1/documents/upload`

### Paramètres de la requête

| Paramètre                | Requis | Type     | Description                                                                                                          |
| ------------------------ | ------ | -------- | -------------------------------------------------------------------------------------------------------------------- |
| file                     | X      | String   | Fichier pdf converti en base64                                                                                       |
| userId                   | X      | String   | Identifiant de l'utilisateur qui émet le document                                                                    |
| profileId ou profileName | X      | String   | Identifiant ou nom du profil à utiliser (détermine le placement du qrCode, les droits d'accès et des tags à ajouter) |
| tagNames[]               |        | [String] | Tableau de noms de tags à ajouter(le tag sera créé si le nom n'existe pas)                                           |

### Réponse du serveur

| Paramètre | Type    | Description                                           |
| --------- | ------- | ----------------------------------------------------- |
| success   | Boolean | Retourne `true` si la requête est traitée avec succès |
| docId     | String  | Identifiant unique du document                        |

# Documents

## Recherche de documents

```shell
curl "https://api.certy.link/v1/documents"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/documents", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("documents");
```

> Le serveur retournera une réponse en JSON de cette forme:

```json
[
  {
    "filename": "FILENAME.pdf",
    "company_id": "COMPANYID",
    "uploadDate": 1551192874541,
    "ownerName": "USERNAME",
    "endDate": 1551192874541,
    "profile_id": "PROFILEID",
    "owner_id": "USERID",
    "id": "DOCUMENTID",
    "tags": ["TAGID1", "TAGID2"],
    "ownerEmail": "USERMAIL"
  },
  {
    "filename": "FILENAME.pdf",
    "company_id": "COMPANYID",
    "uploadDate": 1551192874541,
    "ownerName": "USERNAME",
    "endDate": 1551192874541,
    "profile_id": "PROFILEID",
    "owner_id": "USERID",
    "id": "DOCUMENTID",
    "tags": ["TAGID1", "TAGID2"],
    "ownerEmail": "USERMAIL"
  }
]
```

Ce point d'accès retourne la liste des documents de l'entreprise (limitée à 50 par requête).
Si le paramètre `search` est renseigné, la liste est triée par pertinence, sinon les résultats sont triés par date d'upload (du plus récent au plus ancien).
La liste peut être filtrée par différents paramètres, tous optionnels.

### Requête HTTP

`GET https://api.certy.link/v1/documents`

### Paramètres de la requête

| Paramètre  | Requis | Description                                                                                                                                                                                           |
| ---------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| limit      |        | Limite de résultats par requête (défaut et maximum : 50).                                                                                                                                             |
| start      |        | Curseur de pagination, retourne les résultats à partir de la date d'upload spécifiée au format timestamp unix (défaut : pas de curseur).                                                              |
| user       |        | Identifiant d'utilisateur afin d'utiliser la recherche de document en tant qu'utilisateur, la liste ne pourra retourner que les documents visibles pour l'utilisateur (selon ses droits de lecture).  |
| search     |        | Recherche en texte intégral, le texte spécifié sera recherché dans le nom et le contenu des documents, la liste retournée est triée par pertinence et la pagination par curseur n'est pas disponible. |
| profiles[] |        | Filtre la liste avec un ou plusieurs identifiants de profils, le filtre est inclusif.                                                                                                                 |
| tags[]     |        | Filtre la liste avec un ou plusieurs identifiants de tags, le filtre est exclusif (seuls les documents comprenant tous les tags sont retournés).                                                      |
| startDate  |        | Filtre de date d'upload du document au format timestamp unix, retourne les documents archivés après cette date.                                                                                       |
| endDate    |        | Filtre de date d'upload du document au format timestamp unix, retourne les documents archivés avant cette date.                                                                                       |

<aside class="notice">
Les paramètres des requêtes <code>GET</code> sont à passer à la fin de l'URL après le symbole <code>?</code> (query string parameter) de cette façon :<br />
<code>https://api.certy.link/v1/documents?limit=20&user=USERID&search=TEXTSEARCH&profiles[]=PROFILEID&tags[]=TAGID1&tags[]=TAGID2&startDate=1551192874541</code><br />
(cette requête retourne jusqu'à 20 documents, avec une recherche portée sur le mot clé <code>TEXTSEARCH</code>, la liste est filtrée par le profil <code>PROFILEID</code> et les tags <code>TAGID1</code> et <code>TAGID2</code> et dont la date d'upload est ultérieure au 27 février 2019 à 10:28<br />
</aside>

### Réponse du serveur

La réponse est un tableau contenant des objets avec les propriétés décrites ci dessous :

| Paramètre  | Type     | Description                                                |
| ---------- | -------- | ---------------------------------------------------------- |
| filename   | String   | Nom du fichier                                             |
| company_id | String   | Identifiant de l'entreprise émettrice du document          |
| uploadDate | Number   | Date d'upload du document au format Unix timestamp ms      |
| ownerName  | String   | Nom de l'utilisateur émetteur du document                  |
| endDate    | Number   | Date d'expiration de l'archive au format Unix timestamp ms |
| profile_id | String   | Identifiant du profil de l'archive                         |
| owner_id   | String   | Identifiant de l'utilisateur émetteur du document          |
| id         | String   | Identifiant du document                                    |
| tags       | [String] | Tableau d'identifiants des tags appliqués à l'archive      |
| ownerEmail | String   | Email de l'émetteur du document                            |

## Récupérer un document par son identifiant

```shell
curl "https://api.certy.link/v1/documents/DOCUMENTID"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/documents/DOCUMENTID", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("documents/DOCUMENTID");
```

> Remplacez `DOCUMENTID` par l'identifiant de l'utilisateur recherché

> Le serveur retournera une réponse en JSON de cette forme:

```json
{
  "filename": "FILENAME.pdf",
  "company_id": "COMPANYID",
  "uploadDate": 1551192874541,
  "ownerName": "USERNAME",
  "endDate": 1551192874541,
  "profile_id": "PROFILEID",
  "owner_id": "USERID",
  "id": "DOCUMENTID",
  "tags": ["TAGID1", "TAGID2"],
  "ownerEmail": "USERMAIL"
}
```

Ce point d'accès retourne le détail d'un document correspondant à l'identifiant en paramètre de l'url.

### Requête HTTP

`GET https://api.certy.link/v1/documents/DOCUMENTID`

### Paramètres de la requête

Cette requête ne dispose d'aucun paramètre.

### Réponse du serveur

| Paramètre  | Type     | Description                                                |
| ---------- | -------- | ---------------------------------------------------------- |
| filename   | String   | Nom du fichier                                             |
| company_id | String   | Identifiant de l'entreprise émettrice du document          |
| uploadDate | Number   | Date d'upload du document au format Unix timestamp ms      |
| ownerName  | String   | Nom de l'utilisateur émetteur du document                  |
| endDate    | Number   | Date d'expiration de l'archive au format Unix timestamp ms |
| profile_id | String   | Identifiant du profil de l'archive                         |
| owner_id   | String   | Identifiant de l'utilisateur émetteur du document          |
| id         | String   | Identifiant du document                                    |
| tags       | [String] | Tableau d'identifiants des tags appliqués à l'archive      |
| ownerEmail | String   | Email de l'émetteur du document                            |

## Téléchargement d'un document

```shell
curl "https://api.certy.link/v1/documents/download/DOCUMENTID"
  -H Authorization:'APIKEY'
  -o "FILENAME.pdf"
```

```javascript--browser
const fileName = "FILENAME.pdf";
fetch("https://api.certy.link/v1/documents/download/DOCUMENTID", {
  headers: {
    Authorization: "APIKEY",
  },
}).then(res => {
  return res.blob();
}).then((data) => {
  const a = document.createElement('a');
  document.body.appendChild(a);
  a.style = "display: none";
  const url = window.URL.createObjectURL(data);
  a.href = url;
  a.download = fileName;
  a.click();
  window.URL.revokeObjectURL(url);
});
```

```javascript--node
  const fs = require('fs');

  const filePath = __dirname + '../downloads/FILENAME.pdf');
  const writer = fs.createWriteStream(filePath);
  certyClient.get("documents/download/DOCUMENTID", { responseType: 'stream' })
    .then(response => {
      response.data.pipe(writer); // Ecriture du fichier en stream
      return new Promise((resolve, reject) => {
        writer.on('finish', resolve)
        writer.on('error', reject)
    })
  }).then(() => {
    // Le téléchargement a réussi
  }).catch((err) => {
    // Le téléchargement a échoué
  });
```

> Remplacez FILENAME par le nom du fichier

Ce point d'accès retourne le document spécifié au format PDF

### Requête HTTP

`GET https://api.certy.link/v1/documents/download/DOCUMENTID`

### Paramètres de la requête

Cette requête ne dispose d'aucun paramètre

### Réponse du serveur

La réponse du serveur est un stream de fichier pdf (content-type: application/pdf)

# Utilisateurs

## Récupérer la liste des utilisateurs

```shell
curl "https://api.certy.link/v1/users"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/users", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("users");
```

> Le serveur retournera une réponse en JSON de cette forme:

```json
[
  {
    "id": "USERID",
    "displayName": "USERNAME",
    "email": "EMAIL",
    "admin": true,
    "groups": ["GROUPID1", "GROUPID2"],
    "profiles": {
      "PROFILEID1": "read",
      "PROFILEID2": "write"
    }
  },
  {
    "id": "USERID",
    "displayName": "USERNAME",
    "email": "EMAIL",
    "admin": false,
    "groups": ["GROUPID1", "GROUPID2"],
    "profiles": {
      "PROFILEID1": "read",
      "PROFILEID2": "write"
    }
  }
]
```

Ce point d'accès retourne la liste des utilisateurs de l'entreprise (limitée à 50 par requête).
Les résultats sont triés par email dans l'ordre alphabétique.

### Requête HTTP

`GET https://api.certy.link/v1/users`

### Paramètres de la requête

| Paramètre | Requis | Description                                                                                                            |
| --------- | ------ | ---------------------------------------------------------------------------------------------------------------------- |
| limit     |        | Limite de résultats par requête (défaut et maximum : 50)                                                               |
| start     |        | Curseur de pagination, retourne les résultats dont l'email commence par la valeur de `start` (défaut : pas de curseur) |

<aside class="notice">
Les paramètres des requêtes <code>GET</code> sont à passer à la fin de l'URL après le symbole <code>?</code> (query string parameter) de cette façon :<br />
<code>https://api.certy.link/v1/users?limit=20&start=test@exemple.com</code><br />
(cette requête retourne les 20 prochains utilisateurs après <code>test@exemple.com</code>)<br />
</aside>

### Réponse du serveur

La réponse est un tableau contenant des objets avec les propriétés décrites ci dessous :

| Paramètre   | Type     | Description                                                                                                                    |
| ----------- | -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| id          | String   | Identifiant unique de l'utilisateur                                                                                            |
| displayName | String   | Nom/Prénom de l'utilisateur                                                                                                    |
| email       | String   | Email de l'utilisateur                                                                                                         |
| admin       | Boolean  | `true` si l'utilisateur dispose des droits d'administration                                                                    |
| groups      | [String] | Tableau d'identifiants de groupes auxquels l'utilisateur appartient                                                            |
| profiles    | Object   | Objet dont chaque propriété est un identifiant de profil avec une valeur "read"(droit de lecture) ou "write"(droit d'écriture) |

## Récupérer un utilisateur par son identifiant

```shell
curl "https://api.certy.link/v1/users/USERID"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/users/USERID", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("users/USERID");
```

> Remplacez `USERID` par l'identifiant de l'utilisateur recherché

> Le serveur retournera une réponse en JSON de cette forme:

```json
{
  "id": "USERID",
  "displayName": "USERNAME",
  "email": "EMAIL",
  "admin": true,
  "groups": ["GROUPID1", "GROUPID2"],
  "profiles": {
    "PROFILEID1": "read",
    "PROFILEID2": "write"
  }
}
```

Ce point d'accès retourne le détail de l'utilisateur correspondant à l'identifiant en paramètre de l'url.

### Requête HTTP

`GET https://api.certy.link/v1/users/USERID`

### Paramètres de la requête

Cette requête ne dispose d'aucun paramètre.

### Réponse du serveur

| Paramètre   | Type     | Description                                                                                                                    |
| ----------- | -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| id          | String   | Identifiant unique de l'utilisateur                                                                                            |
| displayName | String   | Nom/Prénom de l'utilisateur                                                                                                    |
| email       | String   | Email de l'utilisateur                                                                                                         |
| admin       | Boolean  | `true` si l'utilisateur dispose des droits d'administration                                                                    |
| groups      | [String] | Tableau d'identifiants de groupes auxquels l'utilisateur appartient                                                            |
| profiles    | Object   | Objet dont chaque propriété est un identifiant de profil avec une valeur "read"(droit de lecture) ou "write"(droit d'écriture) |

# Profils

## Récupérer la liste des profils

```shell
curl "https://api.certy.link/v1/profiles"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/profiles", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("profiles");
```

> Le serveur retournera une réponse en JSON de cette forme:

```json
[
  {
    "id": "PROFILEID1",
    "conservationDuration": 20,
    "name": "PROFILENAME",
    "tags": ["TAGID1", "TAGID2"]
  },
  {
    "id": "PROFILEID2",
    "conservationDuration": 20,
    "name": "PROFILENAME",
    "tags": ["TAGID1", "TAGID2"]
  }
]
```

Ce point d'accès retourne la liste des profils de l'entreprise (limitée à 50 par requête).
Les résultats sont triés par nom dans l'ordre alphabétique.

### Requête HTTP

`GET https://api.certy.link/v1/profiles`

### Paramètres de la requête

| Paramètre | Requis | Description                                                                                                           |
| --------- | ------ | --------------------------------------------------------------------------------------------------------------------- |
| limit     |        | Limite de résultats par requête (défaut et maximum : 50)                                                              |
| start     |        | Curseur de pagination, retourne les résultats dont le nom commence par la valeur de `start` (défaut : pas de curseur) |

### Réponse du serveur

La réponse est un tableau contenant des objets avec les propriétés décrites ci dessous :

| Paramètre            | Type     | Description                                                                             |
| -------------------- | -------- | --------------------------------------------------------------------------------------- |
| id                   | String   | Identifiant unique du profil                                                            |
| conservationDuration | Number   | Durée de conservation des documents archivés avec ce profil en années                   |
| name                 | String   | Nom du profil                                                                           |
| tags                 | [String] | Tableau d'identifiants de tags qui sont ajoutés à chaque document archivé par ce profil |
| qrCode               | Boolean  | Si vrai, un qrCode sera placé sur le document au moment de l'upload                     |

## Récupérer le détail d'un profil par son identifiant

```shell
curl "https://api.certy.link/v1/profiles/PROFILEID"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/profiles/PROFILEID", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("profiles/PROFILEID");
```

> Remplacez `PROFILEID` par l'identifiant du profil recherché

> Le serveur retournera une réponse en JSON de cette forme:

```json
{
  "id": "PROFILEID",
  "conservationDuration": 20,
  "name": "PROFILENAME",
  "tags": ["TAGID1", "TAGID2"],
  "qrCode": true
}
```

Ce point d'accès retourne le détail du profil correspondant à l'identifiant en paramètre de l'url.

### Requête HTTP

`GET https://api.certy.link/v1/profiles/PROFILEID`

### Paramètres de la requête

Cette requête ne dispose d'aucun paramètre.

### Réponse du serveur

| Paramètre            | Type     | Description                                                                             |
| -------------------- | -------- | --------------------------------------------------------------------------------------- |
| id                   | String   | Identifiant unique du profil                                                            |
| conservationDuration | Number   | Durée de conservation des documents archivés avec ce profil en années                   |
| name                 | String   | Nom du profil                                                                           |
| tags                 | [String] | Tableau d'identifiants de tags qui sont ajoutés à chaque document archivé par ce profil |
| qrCode               | Boolean  | Si vrai, un qrCode sera placé sur le document au moment de l'upload                     |

## Récupérer le détail d'un profil par son nom

```shell
curl "https://api.certy.link/v1/profiles/name/PROFILENAME"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/profiles/name/PROFILENAME", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("profiles/name/PROFILENAME");
```

> Remplacez `PROFILENAME` par le nom du profil recherché

> Le serveur retournera une réponse en JSON de cette forme:

```json
{
  "id": "PROFILEID",
  "conservationDuration": 20,
  "name": "PROFILENAME",
  "tags": ["TAGID1", "TAGID2"],
  "qrCode": true
}
```

Ce point d'accès retourne le détail du profil correspondant au nom en paramètre de l'url.

### Requête HTTP

`GET https://api.certy.link/v1/profiles/name/PROFILENAME`

### Paramètres de la requête

Cette requête ne dispose d'aucun paramètre.

### Réponse du serveur

| Paramètre            | Type     | Description                                                                             |
| -------------------- | -------- | --------------------------------------------------------------------------------------- |
| id                   | String   | Identifiant unique du profil                                                            |
| conservationDuration | Number   | Durée de conservation des documents archivés avec ce profil en années                   |
| name                 | String   | Nom du profil                                                                           |
| tags                 | [String] | Tableau d'identifiants de tags qui sont ajoutés à chaque document archivé par ce profil |

# Tags

## Récupérer la liste des tags

```shell
curl "https://api.certy.link/v1/tags"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/tags", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("tags");
```

> Le serveur retournera une réponse en JSON de cette forme:

```json
[
  {
    "id": "TAGID1",
    "color": "#000000",
    "name": "TAGNAME1"
  },
  {
    "id": "TAGID2",
    "color": "#000000",
    "name": "TAGNAME2"
  }
]
```

Ce point d'accès retourne la liste des tags de l'entreprise (limitée à 50 par requête).
Les résultats sont triés par nom dans l'ordre alphabétique.

### Requête HTTP

`GET https://api.certy.link/v1/tags`

### Paramètres de la requête

| Paramètre | Requis | Description                                                                                                           |
| --------- | ------ | --------------------------------------------------------------------------------------------------------------------- |
| limit     |        | Limite de résultats par requête (défaut et maximum : 50)                                                              |
| start     |        | Curseur de pagination, retourne les résultats dont le nom commence par la valeur de `start` (défaut : pas de curseur) |

### Réponse du serveur

La réponse est un tableau contenant des objets avec les propriétés décrites ci dessous :

| Paramètre | Type   | Description                   |
| --------- | ------ | ----------------------------- |
| id        | String | Identifiant unique du tag     |
| color     | String | Couleur du tag en hexadécimal |
| name      | String | Nom du tag                    |

## Récupérer le détail d'un tag par son identifiant

```shell
curl "https://api.certy.link/v1/tags/TAGID"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/tags/TAGID", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("tags/TAGID");
```

> Remplacez `TAGID` par l'identifiant du tag recherché

> Le serveur retournera une réponse en JSON de cette forme:

```json
{
  "id": "TAGID",
  "color": "#000000",
  "name": "TAGNAME"
}
```

Ce point d'accès retourne le détail du tag correspondant à l'identifiant en paramètre de l'url.

### Requête HTTP

`GET https://api.certy.link/v1/tags/TAGID`

### Paramètres de la requête

Cette requête ne dispose d'aucun paramètre.

### Réponse du serveur

| Paramètre | Type   | Description                   |
| --------- | ------ | ----------------------------- |
| id        | String | Identifiant unique du tag     |
| color     | String | Couleur du tag en hexadécimal |
| name      | String | Nom du tag                    |

## Récupérer le détail d'un tag par son nom

```shell
curl "https://api.certy.link/v1/tags/name/TAGNAME"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/tags/name/TAGNAME", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("tags/name/TAGNAME");
```

> Remplacez `TAGNAME` par le nom du tag recherché

> Le serveur retournera une réponse en JSON de cette forme:

```json
{
  "id": "TAGID",
  "color": "#000000",
  "name": "TAGNAME"
}
```

Ce point d'accès retourne le détail du tag correspondant à l'identifiant en paramètre de l'url.

### Requête HTTP

`GET https://api.certy.link/v1/tags/name/TAGNAME`

### Paramètres de la requête

Cette requête ne dispose d'aucun paramètre.

### Réponse du serveur

| Paramètre | Type   | Description                   |
| --------- | ------ | ----------------------------- |
| id        | String | Identifiant unique du tag     |
| color     | String | Couleur du tag en hexadécimal |
| name      | String | Nom du tag                    |

# Groupes

## Récupérer la liste des groupes

```shell
curl "https://api.certy.link/v1/groups"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/groups", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("groups");
```

> Le serveur retournera une réponse en JSON de cette forme:

```json
[
  {
    "id": "GROUPID1",
    "name": "GROUPNAME1",
    "profiles": {
      "PROFILEID1": "read",
      "PROFILEID2": "write"
    },
    "users": ["USERID1", "USERID2"]
  },
  {
    "id": "GROUPID2",
    "name": "GROUPNAME2",
    "profiles": {
      "PROFILEID1": "read",
      "PROFILEID2": "write"
    },
    "users": ["USERID1", "USERID2"]
  }
]
```

Ce point d'accès retourne la liste des groupes de l'entreprise (limitée à 50 par requête).
Les résultats sont triés par nom dans l'ordre alphabétique.

### Requête HTTP

`GET https://api.certy.link/v1/groupes`

### Paramètres de la requête

| Paramètre | Requis | Description                                                                                                           |
| --------- | ------ | --------------------------------------------------------------------------------------------------------------------- |
| limit     |        | Limite de résultats par requête (défaut et maximum : 50)                                                              |
| start     |        | Curseur de pagination, retourne les résultats dont le nom commence par la valeur de `start` (défaut : pas de curseur) |

### Réponse du serveur

La réponse est un tableau contenant des objets avec les propriétés décrites ci dessous :

| Paramètre | Type     | Description                                                                                                                    |
| --------- | -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| id        | String   | Identifiant unique du groupe                                                                                                   |
| name      | String   | Nom du tag                                                                                                                     |
| profiles  | Object   | Objet dont chaque propriété est un identifiant de profil avec une valeur "read"(droit de lecture) ou "write"(droit d'écriture) |
| users     | [String] | Tableau d'identifiants d'utilisateurs                                                                                          |

## Récupérer le détail d'un groupe par son identifiant

```shell
curl "https://api.certy.link/v1/groups/GROUPID"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/groups/GROUPID", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("groups/GROUPID");
```

> Remplacez `GROUPID` par l'identifiant du groupe recherché

> Le serveur retournera une réponse en JSON de cette forme:

```json
{
  "id": "GROUPID",
  "color": "#000000",
  "name": "GROUPNAME"
}
```

Ce point d'accès retourne le détail du groupe correspondant à l'identifiant en paramètre de l'url.

### Requête HTTP

`GET https://api.certy.link/v1/groups/GROUPID`

### Paramètres de la requête

Cette requête ne dispose d'aucun paramètre.

### Réponse du serveur

| Paramètre | Type     | Description                                                                                                                    |
| --------- | -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| id        | String   | Identifiant unique du groupe                                                                                                   |
| name      | String   | Nom du tag                                                                                                                     |
| profiles  | Object   | Objet dont chaque propriété est un identifiant de profil avec une valeur "read"(droit de lecture) ou "write"(droit d'écriture) |
| users     | [String] | Tableau d'identifiants d'utilisateurs                                                                                          |

## Récupérer le détail d'un groupe par son nom

```shell
curl "https://api.certy.link/v1/groups/name/GROUPNAME"
  -H Authorization:'APIKEY'
```

```javascript--browser
fetch("https://api.certy.link/v1/groups/name/GROUPNAME", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
});
```

```javascript--node
certyClient.get("groups/name/GROUPNAME");
```

> Remplacez `GROUPNAME` par le nom du groupe recherché

> Le serveur retournera une réponse en JSON de cette forme:

```json
{
  "id": "GROUPID",
  "color": "#000000",
  "name": "GROUPNAME"
}
```

Ce point d'accès retourne le détail du groupe correspondant à l'identifiant en paramètre de l'url.

### Requête HTTP

`GET https://api.certy.link/v1/groups/name/GROUPNAME`

### Paramètres de la requête

Cette requête ne dispose d'aucun paramètre.

### Réponse du serveur

| Paramètre | Type     | Description                                                                                                                    |
| --------- | -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| id        | String   | Identifiant unique du groupe                                                                                                   |
| name      | String   | Nom du tag                                                                                                                     |
| profiles  | Object   | Objet dont chaque propriété est un identifiant de profil avec une valeur "read"(droit de lecture) ou "write"(droit d'écriture) |
| users     | [String] | Tableau d'identifiants d'utilisateurs                                                                                          |
