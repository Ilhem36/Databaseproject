# Databaseproject
ProjetSQL/Postgres_M1BIBS_2020
--Création du schéma et définition comme chemin par défaut
CREATE SCHEMA projet_caserne;
SET SCHEMA 'projet_caserne';

--Création du type habitation
CREATE TYPE habitation AS ENUM  ('Caserne', 'Ferme','HLM', 'Pavillon');

--Création des tables
---#Création de la table ville
CREATE TABLE Ville (
    
    Nom_ville       VARCHAR(20),
    CP              INTEGER CHECK (CP BETWEEN 1000 AND 98890),
    Nb_hab          INTEGER CHECK ( Nb_hab > 0) NOT NULL ,   
    PRIMARY KEY (Nom_ville,CP)
);

---#Création de la table adresse
CREATE TABLE Adresse (
    
    Num_rue         INTEGER  CHECK (Num_rue>0),
    Nom_rue         VARCHAR(20),
    CP              INTEGER ,
    Nom_ville       VARCHAR(20),
    Type_habitation habitation NOT NULL,
    Proche_caserne  INTEGER ,
    Km              INTEGER NOT NULL,
    FOREIGN KEY (Nom_ville,CP ) REFERENCES Ville (Nom_ville,CP),
    PRIMARY KEY( Num_rue , Nom_rue ,CP,Nom_ville)
);  

---#Création de la table caserne      
CREATE TABLE Caserne (

    Id_caserne      INTEGER ,
    Capa_camion     INTEGER  CHECK ( Capa_camion >0) NOT NULL,
    Capa_pompier    INTEGER  CHECK  (Capa_pompier>0) NOT NULL, 
    Nom_rue         VARCHAR(20) NOT NULL ,
    CP              INTEGER NOT NULL,
    Nom_ville       VARCHAR(20) NOT NULL, 
    Num_rue         INTEGER NOT NULL,
    FOREIGN KEY (Num_rue,Nom_rue,CP, Nom_ville ) REFERENCES Adresse (Num_rue,Nom_rue,CP, Nom_ville),   
    PRIMARY KEY (Id_caserne)
);

--- Ajout de la clé étrangère 'Proche_caserne' de la table Adresse référant à Id_caserne de la table Caserne
ALTER TABLE Adresse ADD CONSTRAINT adresse_proche_caserne_fkey FOREIGN KEY (Proche_caserne ) REFERENCES Caserne (Id_caserne );

---#Création de la table protège
CREATE TABLE Protege (
    Id_caserne      INTEGER ,
    Nom_ville       VARCHAR(20),
    CP              INTEGER ,
    FOREIGN KEY (Id_caserne) REFERENCES Caserne (Id_caserne),
    FOREIGN KEY (Nom_ville,CP ) REFERENCES Ville (Nom_ville,CP),
    PRIMARY KEY (Id_caserne,Nom_ville, CP)
);

--#Création de la table pompier            
CREATE TABLE Pompier (

    Id_caserne      INTEGER,
    Id_pompier      INTEGER ,
    Nom             VARCHAR(20) NOT NULL ,
    Prenom          VARCHAR(20) NOT NULL ,
    Nom_rue         VARCHAR(20) NOT NULL ,
    Num_rue         INTEGER NOT NULL,
    Nom_ville       VARCHAR(20) NOT NULL , 
    CP              INTEGER NOT NULL,
    FOREIGN KEY (Num_rue,Nom_rue,CP, Nom_ville ) REFERENCES Adresse (Num_rue,Nom_rue,CP, Nom_ville),
    FOREIGN KEY (Id_caserne) REFERENCES Caserne (Id_caserne ),
    PRIMARY KEY (Id_caserne,Id_pompier)
);

--#Création de la table fabricant
CREATE TABLE Fabricant  (

    Nom_fabricant   VARCHAR(20),
    Delai           INTEGER NOT NULL,     
    Num_rue         INTEGER NOT NULL,
    Nom_rue         VARCHAR(20) NOT NULL,
    CP              INTEGER NOT NULL,
    Nom_ville       VARCHAR(20) NOT NULL, 
    FOREIGN KEY (Num_rue,Nom_rue,CP, Nom_ville ) REFERENCES Adresse (Num_rue,Nom_rue,CP, Nom_ville),
    PRIMARY KEY (Nom_fabricant)
);

---#Création de la table modèle
CREATE TABLE Modele   (
    Nom_modele      VARCHAR(20),
    Type_modele     VARCHAR(20) NOT NULL ,  
    Motorisation    VARCHAR(20) NOT NULL ,
    Nom_fabricant   VARCHAR(20) NOT NULL , 
    FOREIGN KEY (Nom_fabricant) REFERENCES  Fabricant (Nom_fabricant),
    PRIMARY KEY ( Nom_modele ) 
);

---#Création de la table camion
CREATE TABLE Camion     (
    Id_caserne      INTEGER ,
    Id_camion       INTEGER ,
    Nb_places       INTEGER  NOT NULL ,
    Modele          VARCHAR(20) NOT NULL,
    FOREIGN KEY ( Modele) REFERENCES Modele (Nom_modele),
    FOREIGN KEY (Id_caserne ) REFERENCES Caserne (Id_caserne ),
    PRIMARY KEY ( Id_caserne, Id_camion)   
);

---#Création de la table citerne
CREATE TABLE Citerne    (

    Id_caserne      INTEGER ,
    Id_camion       INTEGER ,
    Contenance      INTEGER NOT NULL ,
    FOREIGN KEY (Id_caserne ,Id_camion) REFERENCES Camion (Id_caserne,Id_camion),
    PRIMARY KEY ( Id_caserne, Id_camion)
);
# Insertion 
    
--Insertion dans la table ville
INSERT INTO Ville (Nom_ville, CP, Nb_hab) VALUES('Montrouge', 92120, 50260);
INSERT INTO Ville VALUES('Malakoff', 92240, 30720);
INSERT INTO Ville VALUES('Châtillon', 92320, 37555);
INSERT INTO Ville VALUES('Rueil-Malmaison',92500, 78152);
INSERT INTO Ville VALUES('Boulogne-Billancourt',92100, 120071);

--Insertion dans la table adresse
INSERT INTO Adresse (Num_rue, Nom_rue, CP, Nom_ville, Type_habitation, Proche_caserne, Km) VALUES(53,'Rue de la Vanne',92120,'Montrouge','Caserne',NULL,0);
INSERT INTO Adresse VALUES(102,'Avenue de Pompei', 92100,'Boulogne-Billancourt','Caserne',NULL,0);
INSERT INTO Adresse VALUES(15,'Rue André Coin',92240,'Malakoff','Pavillon',NULL,2);
INSERT INTO Adresse VALUES(2,'Allée Berlioz',92320,'Châtillon','HLM',NULL,3);
INSERT INTO Adresse VALUES(7, 'rue de la voiture', 92500,'Rueil-Malmaison', 'Ferme', NULL, 02);
INSERT INTO Adresse VALUES(13,'quai du camion',92100,'Boulogne-Billancourt','Ferme', NULL, 04);

--Insertion dans la table caserne
INSERT INTO Caserne (Id_caserne, Capa_camion, Capa_pompier, Nom_rue, CP, Nom_ville, Num_rue) VALUES(02,10,50,'Rue de la Vanne',92120,'Montrouge',53);
INSERT INTO Caserne VALUES(45,3,16,'Avenue de Pompei', 92100,'Boulogne-Billancourt',102);

--Mise à jour de la table Adresse pour récupérer les valeurs de 'Proche_caserne' à partir du code postal de l'adresse
UPDATE Adresse SET Proche_caserne = Protege.Id_caserne FROM Protege WHERE Adresse.CP = Protege.CP AND Adresse.Nom_Ville = Protege.Nom_Ville;

--Insertion dans la table protege
INSERT INTO Protege (Id_caserne, Nom_ville,CP) VALUES(02, 'Châtillon', 92320);
INSERT INTO Protege VALUES(02, 'Montrouge', 92120);
INSERT INTO Protege VALUES(45, 'Boulogne-Billancourt', 92100);
INSERT INTO Protege VALUES(02, 'Malakoff', 92240);
INSERT INTO Protege VALUES(02, 'Rueil-Malmaison',92500);

--Insertion dans la table pompier
INSERT INTO Pompier (Id_caserne,Id_pompier, Nom, Prenom, Nom_rue, Num_rue, Nom_ville, CP) VALUES(02, 0215, 'DUPONT', 'Robert','Rue André Coin',15,'Malakoff',92240);
INSERT INTO Pompier VALUES(02, 0218, 'SCHNEIDER', 'Sarah', 'Allée Berlioz', 2,'Châtillon',92320);


--Insertion dans la table fabricant
INSERT INTO Fabricant (Nom_fabricant, Delai, Num_rue, Nom_rue, CP, Nom_ville) VALUES('Renault',15 ,13,'quai du camion',92100,'Boulogne-Billancourt');
INSERT INTO Fabricant VALUES('Citroën',10,7, 'rue de la voiture', 92500,'Rueil-Malmaison');

--Insertion dans la table modele
INSERT INTO Modele (Nom_modele, Type_modele, Motorisation, Nom_fabricant) VALUES('85 150 TI','lourd','150ch','Renault');
INSERT INTO Modele VALUES('Premium210','lourd','210ch','Renault');

--Insertion dans la table camion
INSERT INTO Camion (Id_caserne, Id_camion, Nb_places, Modele) VALUES(02,0258,4,'85 150 TI');
INSERT INTO Camion VALUES(45,4551,2,'Premium210');

--Insertion dans la table citerne
INSERT INTO Citerne (Id_caserne,Id_camion,Contenance) VALUES(02,0258,2000);
INSERT INTO Citerne VALUES(45,4551,3500);

--Insertions non respectueuses de contraintes
INSERT INTO Ville VALUES ('Malakoff', 92240, 33720); /*ne respecte pas clé primairem, car Malakoff est déja prése-- Q1: Casernes protégeant Brignoles et Le Luc et leur ville
# Requetes 
SELECT Cas.id_caserne, Cas.nom_ville, Cas.cp
    FROM caserne Cas, protege Pr
    WHERE Cas.id_caserne = Pr. id_caserne
      AND Pr.nom_ville = 'Brignoles'
INTERSECT
SELECT Cas.id_caserne, Cas.nom_ville, Cas.cp
    FROM caserne Cas, protege Pr
    WHERE Cas.id_caserne = Pr. id_caserne
      AND Pr.nom_ville = 'Le Luc';

--Q2 : Id, noms et prénoms des pompiers de la caserne 3 habitant à plus de 5km d'une caserne
SELECT Po.id_pompier, Po.nom, Po.prenom
    FROM pompier Po, adresse A
    WHERE A.nom_rue = Po.nom_rue AND A.num_rue = Po.num_rue
      AND A.nom_ville = Po.nom_ville AND A.cp = Po.cp
      AND Po.id_caserne = 3 AND A.km > 5;

--Q3 id, noms et prénoms des pompiers habitant Le Luc ou des villes >= 20 000 habitants
/*On fait une union entre les tables des pompiers habitant à Le Luc 
  et ceux habitant dans une ville avec plus de 20 000 habitant*/
SELECT  Po.id_pompier, Po.nom, Po.prenom
    FROM pompier Po, ville V
    WHERE Po.nom_ville = V.nom_ville AND Po.cp = V.cp
      AND Po.nom_ville = 'Le Luc'
UNION
SELECT  Po.id_pompier, Po.nom, Po.prenom
    FROM pompier Po, ville V
    WHERE Po.nom_ville = V.nom_ville AND Po.cp = V.cp
      AND V.nb_hab >= 20000;

--Q4 Délai moyen de livraison des fabriquants de citernes de moins de 1000 litres
/*ici on utilise une sous requete pour ne pas tenir compte dans le calcul de la
  moyenne, des doublons de fabricants (certains fabricants fabriquent plusieurs
  modeles de citernes et pour plusieurs casernes, donc certaines lignes sont dedoublées)*/
SELECT AVG(F.delai) AS "Delai moyen"
    FROM fabricant F
    WHERE F.nom_fabricant in
          (SELECT F.nom_fabricant
            FROM fabricant F, modele M,  Camion Cam, citerne Ci
            WHERE F.nom_fabricant = M.nom_fabricant AND M.nom_modele = Cam.modele
              AND Cam.id_caserne = Ci.id_caserne AND Cam.id_camion = Ci.id_camion
              AND Ci.contenance < 1000);

--Q5 Temps moyen de livraison des camions par caserne en ordre décroissant
SELECT Cas.id_caserne, AVG(F.delai) AS "Temps moyen de livraison"
    FROM fabricant F, modele M, camion Cam, caserne Cas
    WHERE F.nom_fabricant = M.nom_fabricant
      AND M.nom_modele = Cam.modele
      AND Cam.id_caserne = Cas.id_caserne
    GROUP BY Cas.id_caserne
    ORDER BY AVG(F.delai) DESC;

--Q6 Nombre de pompiers par caserne
SELECT Cas.id_caserne ,COUNT(Po.id_pompier) AS "Nombre de pompiers"
    FROM pompier Po, caserne Cas
    WHERE Po.id_caserne = Cas.id_caserne
    GROUP BY Cas.id_caserne ;

--Q7 Id et ville des casernes avec citerne avec le plus de contenance
SELECT Cas.id_caserne, Cas.nom_ville, Cas.cp
    FROM caserne Cas, citerne Ci
    WHERE Cas.id_caserne = Ci.id_caserne
      AND ci.contenance = (SELECT MAX(contenance)
                              FROM citerne Ci);

--Q8 Casernes ayant atteint leur capacité maximale humaine
SELECT Po.id_caserne
    FROM caserne Cas, pompier Po
    WHERE Po.id_caserne = Cas.id_caserne
    GROUP BY Po.id_caserne, Cas.capa_pompiers
    HAVING Cas.capa_pompiers = COUNT(distinct (Po.id_caserne,Po.id_pompier));

--Q9 Id, nom, prenom ne travaillant pas dans la ville où ils habitent
/*On prend l'ensemble des pompiers et on enlève ceux qui travaillent où ils habitent*/
SELECT Po.id_caserne, Po.id_pompier, Po.nom, Po.prenom,
       Po.nom_ville AS "Ville d'habitation", Po.cp AS "CP d'habitation",
       Cas.nom_ville AS "Ville de caserne",Cas.cp AS "CP de caserne"
    FROM caserne Cas, pompier Po
    WHERE Po.id_caserne = Cas.id_caserne
EXCEPT
SELECT  Po.id_caserne, Po.id_pompier, Po.nom, Po.prenom,
        Po.nom_ville, Po.cp,
        Cas.nom_ville,Cas.cp
    FROM caserne Cas, pompier Po
    WHERE Po.id_caserne = Cas.id_caserne
      AND Po.nom_ville = Cas.nom_ville AND Po.cp=Cas.cp;

--Q10 Liste décroissante des casernes en fonction du nombre de pompiers y travaillant
SELECT Cas.id_caserne ,COUNT(Po.id_pompier) AS "Nombre de pompiers"
    FROM caserne cas, pompier Po
    WHERE Po.id_caserne= Cas.id_caserne
    GROUP BY Cas.id_caserne
    ORDER BY COUNT(Po.id_pompier) DESC ;

--Q11 Première caserne de la liste précédente
SELECT Cas.id_caserne ,COUNT(Po.id_pompier) AS "Nombre de pompiers"
    FROM caserne cas, pompier Po
    WHERE Po.id_caserne= Cas.id_caserne
    GROUP BY Cas.id_caserne
    ORDER BY COUNT(Po.id_pompier) DESC
    LIMIT 1;

--Q12 Volume d' eau total des citernes pour chaque caserne
SELECT Cas.id_caserne, SUM(Ci.contenance) AS "Volume en eau total"
    FROM caserne Cas, citerne Ci
    WHERE Cas.id_caserne = Ci.id_caserne
    GROUP BY Cas.id_caserne;

--Q13 Caserne sans citerne
/*On prend l'ensemble des casernes et on enlève les casernes avec citerne*/
SELECT Cas.id_caserne
    FROM caserne Cas
EXCEPT
SELECT Cas.id_caserne
    FROM caserne Cas, citerne Ci
    WHERE Cas.id_caserne= Ci.id_caserne;

--Q14 Villes  protégées par au moins deux casernes
/*On utilise un produit cartésien (Pr1 X Pr2)pour trouver deux villes différentes
  protégeant une meme ville*/
SELECT DISTINCT Pr1.nom_ville, Pr1.cp
    FROM protege Pr1, protege Pr2
    WHERE Pr1.nom_ville = Pr2.nom_ville AND Pr1.cp = Pr2.cp
      AND Pr1.id_caserne != Pr2.id_caserne;

--Q15 Nb d'habitants moyen --Suppression des tables
DROP TABLE Citerne;
DROP TABLE Camion;
DROP TABLE Modele;
DROP TABLE Fabricant;
DROP TABLE Protege;
DROP TABLE Pompier;
ALTER TABLE Adresse DROP CONSTRAINT adresse_proche_caserne_fkey; /*suppression de la clé étrangère 'Proche_caserne' qui empêche la suppression de la table caserne*/
DROP TABLE Caserne;
DROP TABLE Adresse;
DROP TABLE Ville;
--Suppression du type
DROP TYPE habitation;
# Suppression 
--Suppression du schéma
DROP SCHEMA projet_caserne;
dans les villes protégées par des casernes de plus de deux camions
/*ici on utilise une sous requete pour ne pas tenir compte dans le calcul de la
  moyenne, des doublons de villes (certaines villes sont protegees par plusieurs casernes)*/
SELECT AVG(V.nb_hab)
FROM Ville V
WHERE (V.nom_ville,V.cp) in (SELECT V.nom_ville,V.cp
    FROM ville V, protege Pr, camion Cam
    WHERE V.nom_ville = Pr.nom_ville AND V.cp=Pr.cp
      AND Pr.id_caserne = Cam.id_caserne
    GROUP BY pr.id_caserne, V.cp, V.nom_ville
    HAVING COUNT(DISTINCT(Cam.id_caserne,Cam.id_camion)) > 2);
nt avec un nombre d‘habitants différent*/
INSERT INTO Citerne VALUES(45,4553,3500); /*ne respecte pas clé étrangère, car il n‘y a pas de camion 4553*/
INSERT INTO Ville VALUES ('Beijing',100000,21542000); /*ne respecte pas contrainte sur le code postal */

