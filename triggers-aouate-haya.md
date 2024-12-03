## Compte rendu

### 1. Trigger de Mise à Jour de la Disponibilité des Voitures lors d'une Réservation

- **Nom du Trigger** : `mis_a_jour_voitDispo`
- **Événement** : `AFTER INSERT` sur la table `Réservations`
- **Objectif** :
    - Mettre à jour la disponibilité des voitures lors d'une nouvelle réservation oar un client

#### Code SQL :

```sql
DELIMITER //
CREATE TRIGGER  mis_a_jour_voitDispo
    AFTER INSERT ON Réservations
    FOR EACH ROW
BEGIN
    UPDATE Voitures
    SET disponible = 0
    WHERE id = NEW.voiture_id;
end ;

```

### 2. Trigger de Vérification de la validité de l'age d'un nouveau client

- **Nom du Trigger** : `verif_age`
- **Événement** : `BEFORE INSERT` sur la table `Clients`
- **Objectif** :
- Verifier avant d'inserer un nouveau client que son age est superieur a 20 ans

#### Code SQL :

```sql  

DELIMITER //
CREATE TRIGGER  verif_age
    BEFORE INSERT ON Clients
    FOR EACH ROW
BEGIN
    if (NEW.age < 21) then
            signal sqlstate '45000' set message_text = 'vous devez avoir au moins 21 ans';
end if ;
end ;

```  
### 3. Verifier la validité du permis de conduire d'un nouveau client

- **Nom du Trigger** : `verif_validite_permiss`
- **Événement** : `BEFORE INSERT` sur la table `Clients`
- **Objectif** :Verifier avant d'inserer un nouveau client que son permis de conduire est valide.
  C'est a dire qu'il ne soit pas deja dans la base de donnees (unique), qu'il inscrit obligatoirement un permis de conduire.
- Il faut qu'il soit composé uniquement de lettre ,de chiffres et soit d'une longueur de 15 caractères.

#### Code SQL :

```sql  


DELIMITER //
CREATE TRIGGER verif_validite_permiss
    BEFORE INSERT ON Clients
    FOR EACH ROW
    BEGIN
        IF EXISTS (
            SELECT 1
            FROM Clients
            WHERE permis_conduire= NEW.permis_conduire
        ) THEN
            SIGNAL SQLSTATE '45000'
                SET MESSAGE_TEXT = 'Le permis de conduire doit être unique.';
        END IF;
         IF LENGTH(NEW.permis_conduire) <> 15 THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Le permis de conduire doit contenir 15 caractères.';
    END IF;
        IF NEW.permis_conduire NOT REGEXP '^[A-Z0-9]{15}$' THEN
            SIGNAL SQLSTATE '45000'
                SET MESSAGE_TEXT = 'Le permis de conduire doit être composé uniquement de caractères alphanumériques.';
        END IF;
        IF NEW.permis_conduire IS NULL OR NEW.permis_conduire = '' THEN
            SIGNAL SQLSTATE '45000'
                SET MESSAGE_TEXT = 'Un permis de conduire est obligatoire pour lenrengistrement.';
        END IF;
    END;
```
### 4.Verification disponibilité voiture

- **Nom du Trigger** : `verif_voiture_dispo`
- **Événement** : `BEFORE INSERT` sur la table `Réservations`
- **Objectif** : Verifier avant d'inserer une nouvelle reservation que la voiture est disponible.

#### Code SQL :

```sql
DELIMITER //
CREATE TRIGGER verif_voiture_dispo
    BEFORE insert on Réservations
    FOR EACH ROW
    BEGIN
        DECLARE voiture_disponible BOOLEAN;

        SELECT disponible INTO voiture_disponible
        FROM Voitures
        WHERE id = NEW.voiture_id;

        IF voiture_disponible = 0 THEN
            SIGNAL SQLSTATE '45000'
                SET MESSAGE_TEXT = 'La voiture demandée n’est pas disponible pour réservation.';
        END IF;
    end //

```

### 5.Eviter chevauchement de reservation

- **Nom du Trigger** : `verif_reservation_non_chevauche`
- **Événement** : `BEFORE INSERT` sur la table `Réservations`
- **Objectif** : Eviter les chevauchement de reservation sur la meme voiture.
- Si une reservation est chevauché, un message d'erreur doit etre renvoyé.

#### Code SQL :

```sql  
DELIMITER //
CREATE TRIGGER verif_reservation_non_chevauche
    BEFORE INSERT ON Réservations
    FOR EACH ROW
    BEGIN
        IF EXISTS (
        SELECT 1
        FROM Réservations
        WHERE voiture_id = NEW.voiture_id
          AND (
            (NEW.date_debut < date_fin AND NEW.date_fin > date_debut) 
        OR (NEW.date_debut = date_debut AND NEW.date_fin = date_fin)
        OR (NEW.date_debut >= date_debut AND NEW.date_fin <= date_fin)
        ) 
        )
        THEN
        SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'La voiture est déjà réservée pour cette période.';
    END IF;
    END;

```
