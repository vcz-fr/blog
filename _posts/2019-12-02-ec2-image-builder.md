---
layout: blog

title: "EC2 Image builder: la construction des images EC2 selon AWS"
categories: ["Cloud"]
tags: ["repost", "gekko", "cloud", "aws", "re:invent"]

date: "2019-12-02"
---

**Words of notice:** This is an article I wrote for [Gekko](https://www.gekko.fr/). Many thanks for their consent to
sharing this story here.

***

_Gekko @ re:Invent 2019 – Lundi 2 Décembre 2019_

## Un problème de construction d’images

[Amazon Elastic Compute Cloud](https://aws.amazon.com/ec2/) ou [EC2](https://aws.amazon.com/ec2/) est l’un des services
phares de AWS. Sa portée en termes de besoins couverts est immense et tous les jours grandissante avec la mise à
disposition de nouvelles variantes de matériels, de systèmes d’exploitation et de services connectables.

Quand un service est aussi développé, il devient un standard et forme naturellement son écosystème. Aujourd’hui, cette
nouveauté concerne l’un des points faibles de EC2 qui aura certainement causé des maux de tête à plus d’une personne :
la construction des [Amazon Machine Images](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html).

<!-- READ MORE -->

Prenons l’exemple de [Docker](https://www.docker.com/). Pour créer une image, il est courant de se baser sur un [Dockerfile](https://docs.docker.com/engine/reference/builder/)
que l’on exécute ensuite avec docker build ou bien avec [Buildah](https://buildah.io/). Ce processus exécute une suite
de commandes, potentiellement accompagnées de tests et produit l’image. On peut ensuite créer un environnement, le
“conteneur”, à partir de cette image ou bien l’envoyer dans un système qui l’hébergera et assurera sa distribution, le
“registre”.

Comment cela se traduire-il pour AWS ? Jusqu’à présent, il fallait mettre en place un processus qui démarre une instance
EC2, récupère les composants à installer et sauvegarde l’instance. Cela requiert une organisation stricte en raison de
mise en production future de l’AMI et de l’utilisation d’outils d’automatisation tiers comme [Ansible](https://www.ansible.com/)
ou [Packer](https://www.packer.io/). Les solutions tierces ont leurs avantages et leurs inconvénients : le besoin peut
être rempli plus facilement mais la solution ne peut être facilement flexible et causera des frictions opérationnelles
dues à la communication entre outils.

Il manquait donc un composant à AWS permettant d’intégrer ce maillon à la chaîne de production EC2. Vient **EC2 Image
Builder**.

## EC2 Image Builder

Imaginé pour offrir l’éventail le plus large de possibilités, EC2 Image Builder simplifie les étapes nécessaires à la
création d’une AMI du choix de son système d’exploitation, de ses composants logiciels et de ses tests à la
configuration du processus même. En plus de cela, EC2 Image Builder peut automatiser le déclenchement d’un processus de
construction, permettre de rendre les images publiques, les répliquer sur d’autres régions, leur appliquer
automatiquement une licence, des tags, choisir le matériel qui va la compiler, envoyer des notifications de statut et
ainsi de suite.

Cette solution ne serait pas complète si elle n’offrait pas de flexibilité à ses utilisateurs. Bien que EC2 Image
Builder ne supporte que les systèmes d’exploitation Amazon Linux 2, Windows 2012R2, 2016 et 2019, on propose des
composants logiciels pour **créer** des solutions sous PHP, Java –avec [Amazon Corretto](https://aws.amazon.com/corretto/)-,
Docker, Python, les **sécuriser** appliquant des [STIG](https://en.wikipedia.org/wiki/Security_Technical_Implementation_Guide)
et en autorisant les mises à jour du système d’exploitation et les **tester** en vérifiant leur état, leur capacité à
créer des fichiers, se connecter au réseau, redémarrer, utiliser un gestionnaire de paquets ou se connecter avec SSH. De
plus, il est possible d’ajouter ses propres composants afin de créer des solutions basées sur des technologies
différentes, de les sécuriser et de les tester de façon plus variée.

Du fait de son intégration avec AWS, EC2 Image Builder est une solution à considérer lorsque l’on souhaite générer
automatiquement des AMI qui soient toujours à jour pour ses applications. Son intégration avec AWS limite les risques
liés à l’usage de solutions tierces tout en présentant une utilisation raisonnable des ressources. Sans surcoût, seule
l’infrastructure exécutant le processus de construction d’image est facturée.

> Image Builder is offered at no cost, other than the cost of the underlying AWS resources used to create, store, and
share the images.

Couplé à SNS et AWS Lambda, EC2 Image Builder peut compléter une architecture en générant une image applicable à des
groupes d’instances. Le processus de génération d’image peut être déclenché manuellement ou planifié pour une exécution
régulière, ce qui peut être intéressant dans le cadre de l’automatisation de la mise à jour d’une application.

Enfin, il est important de noter que les SDK AWS ont été mis à jour pour supporter AWS Builder : [Python](https://github.com/boto/boto3/blob/develop/.changes/1.10.29.json),
[Go](https://github.com/aws/aws-sdk-go/releases/tag/v1.25.44), [JavaScript](https://github.com/aws/aws-sdk-js/blob/master/CHANGELOG.md#25810),
[.NET](https://github.com/aws/aws-sdk-net/blob/master/SDK.CHANGELOG.md#336400-2019-12-02-0931-utc), [Java](https://github.com/aws/aws-sdk-java/blob/master/CHANGELOG.md#111684-2019-12-02),
[Java v2](https://github.com/aws/aws-sdk-java-v2/blob/master/.changes/2.10.26.json), [PHP](https://github.com/aws/aws-sdk-php/releases/tag/3.124.0)
et [Ruby](https://github.com/aws/aws-sdk-ruby/releases/tag/v2.11.407). Cela laisse présager que les applications qui
dépendent de ces bibliothèques seront prochainement mises à jour à leur tour.

## Démonstration

![Étape 1 : Recette](/assets/img/posts/20191202/step1-recipe.png)

L’interface reprend les codes des services les plus récents. Attention toutefois à ne pas presser le bouton Précédent
par inadvertance : cela a pour effet de réinitialiser le formulaire au retour.

La première étape consiste à définir le contenu de l’AMI à créer et des tests à exécuter. C’est également lors de cette
étape que l’on indique que l’image doit rester à jour par rapport à son image de base.

![Étape 2 : Processus](/assets/img/posts/20191202/step2-pipeline.png)

La deuxième étape de la définition du processus consiste à paramétrer le processus lui-même : les instances EC2 sur
lesquelles il va se baser, sa fréquence, son rôle EC2, son VPC, etc.

Cette étape est très importante car elle pose les bases d’un processus de construction d’AMI sécurisé et dont les accès
sont contrôlés et limités. On rencontre couramment ce schéma de pensée dans les bonnes pratiques liées à l’utilisation
de AWS.

![Étape 3 : Configurations](/assets/img/posts/20191202/step3-additional.png)

Viennent ensuite les configurations additionnelles. Cette étape concerne les paramètres de distribution des images, les
tags à appliquer ainsi que la licence à associer, de manière optionnelle.

On peut ainsi remarquer qu’il est, par exemple, envisageable de dédier un compte AWS d’une organisation à la création
des AMI. Les images peuvent ensuite être automatiquement exposées aux comptes AWS de l’organisation qui souhaitent les
utiliser.

![Étape 4 : Relecture](/assets/img/posts/20191202/step4-review.png)

Enfin, la dernière étape de la définition du processus est une relecture avant la validation. Le processus ne
s’exécutera pas immédiatement après sa création. Sa première exécution suivra ainsi la fréquence qui lui a été assignée.

EC2 Image Builder est proposé dans toutes les régions AWS. Il ne reste plus qu’à [découvrir le service](https://console.aws.amazon.com/imagebuilder/).