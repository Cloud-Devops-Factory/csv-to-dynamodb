# Outil de migration de schéma DynamoDB

## Prémisse majeure
--Étant donné que DynamoDB est une base de données NoSQL sans schéma, vous pouvez librement ajouter des attributs normaux sans modifier la définition de la table.
--Cependant, en tant que partie liée à la définition de la table
   ―― 1. Clé de partition = Clé de hachage
   ―― 2. Clé de tri = Clé de plage
   ―― 3. LSI
   ―― 4. ISG
-Ils sont quatre

La modification des quatre éléments ci-dessus pour une table existante s'appelle la migration de schéma.

## Ajouter / Supprimer GSI
Vous pouvez librement ajouter/supprimer des tables existantes

## Ajouter/supprimer LSI
Ne peut pas être ajouté/supprimé à une table existante

LSI est créé uniquement lorsque la table est créée.

Si vous voulez vraiment ajouter/supprimer, vous devrez recréer le tableau.

Voir : https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/LSI.html#LSI.Creating (`Les index secondaires locaux sur une table sont créés lors de la création de la table.`)

## Modifier la clé de partition et la clé de tri
Ne peut pas être modifié pour une table existante

Si vous voulez vraiment le changer, vous devrez recréer la table.

## Migration de données due à la recréation de la table
Il est nécessaire de recréer la table lors de l'ajout/de la suppression de LSI à la table existante et de la modification de la clé de partition et de la clé de tri.
Dans le cas peu probable où il deviendrait nécessaire de recréer la table en fonctionnement, suivez la procédure ci-dessous.

―― 1. Exportez au format CSV le contenu de l'ancienne table depuis la console de gestion de DynamoDB
―― 2. Supprimez l'ancienne table de la console de gestion et créez une nouvelle table
    --Si vous utilisez Cloudformation, supprimez la partie définition de l'ancienne table de la pile Cloudformation existante et déployez
    --Déployer en ajoutant une nouvelle définition de table
--3. Sélectionnez "Télécharger le fichier de modèle" dans "Créer une pile" dans Cloudformation de la console de gestion.
--4. Chargez `cloudformation / csvToDynamo.template` de ce référentiel, entrez" Nom du compartiment S3 pour le téléchargement CSV nouvellement créé "," Nom de la nouvelle table DynamoDB ", et" Nom du fichier CSV exporté en 1 ". Créez Lambda pour insérer S3 Bucket et CSV chargés dans ce Bucket dans DynamoDB
--5. Chargez le fichier csv de 1. dans le compartiment S3
―― 6. Vérifier le contenu de la nouvelle table de DynamoDB
