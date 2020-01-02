# Errors

<aside class="notice">
Chaque requête peut retourner une erreur pour de multiples raisons, un code erreur sera retourné afin de vous permettre d'identifier l'erreur et d'effectuer les actions appropriées.
</aside>

```shell
curl "https://api.certy.link/v1/documents/upload/base64"
  -X POST
  -H "Authorization: APIKEY"
```

```javascript--browser
fetch("https://api.certy.link/v1/ENDPOINT", {
  headers: {
    "Content-Type": "application/json",
    Authorization: "APIKEY",
  },
})
  .then(res => res.json())
  .then(responseData => {
    if (!responseData.error) {
      // Réponse du serveur, requête traitée avec succès
    } else {
      const errorCode = responseData.error.code; // Code erreur CertyLink
      const errorMessage = responseData.error.message; // Message d'erreur
    }
  })
  .catch(err => {
    // Erreur réseau
  });
```

```javascript--node
certylinkClient.get("ENDPOINT")
    .then(res => {
        // Réponse du serveur, requête traitée avec succès
        const responseData = res.data;
    })
    .catch(err => {
        if (err.response) {
            // Erreur lors du traitement de la requête
            const responseData = err.response.data;
            const errorCode = responseData.error.code; // code erreur CertyLink
            const errorMessage = responseData.error.message; // Message d'erreur
        } else {
            // Erreur réseau
        }
  });
```

Voici la liste des codes d'erreur que vous pouvez rencontrer en utilisant l'API CertyLink pro :

| Error Code            | Error Status | Explications                                                                                   |
| --------------------- | ------------ | ---------------------------------------------------------------------------------------------- |
| incorrect_mimetype    | 400          | Le type de fichier envoyé est incorrect ou non pris en charge (pdf uniquement)                 |
| missing_payload       | 400          | Un paramètre requis est manquant (vérifiez que vos données sont envoyées au bon format)        |
| forbidden             | 403          | Accès refusé, la clé API est erronée ou non spécifiée                                          |
| not_found             | 404          | Mauvaise URL/Méthode ou entitée introuvable                                                    |
| incorrect_payload     | 422          | Un paramètre de la requête est erroné                                                          |
| rate_limit_error      | 429          | Limite de requêtes par minute dépassée, veuillez attendre avant d'envoyer une nouvelle requête |
| internal_server_error | 500          | Le serveur a rencontré une erreur inattendue                                                   |
