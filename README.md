# Internationalisation & localisation avec I18n
***
![I18n](https://www.proactiveinvestors.co.uk/upload/Article/Image/2019_04/1555571446_shutterstock_340405046.jpg)
Cette mission a pour but de vous aider à traduire le plus simplement et efficacement possible vos projets Rails à l'aide de la gemme i18n.


## 1. Présentation
***


I18n et L10n sont les numéronymes respectifs d'Internationalization et de Localization (il y a 18 lettres entre le "I" et le "n" et 10 entre le "L" et le "n"). 
Il est important tout d'abord de bien distinguer leurs différentes fonctions.

### 1.1 I18n
***

  I18n va définir l'ensemble des moyens techniques mis en place pour adapter un contenu associé à une culture à une autre.
  Quand on pense traduction, on pense tout de suite à traduire du texte, mais cela va en réalité beaucoup plus loin : 
  On ne paye pas avec la même monnaie en Europe qu'aux USA, on n'utilise pas non plus le même format de date, on n'écrit 
  pas dans le même sens en arabe, on ne forme pas le pluriel de la même façon ...
  Toutes ces différentes valeurs, que ce soit $ ou €, %month%day%year ou %day%month%year ... vont se retrouver liées à 
  des clefs qui viendront remplacer les valeurs "en dur" écrites dans le code HTML.
   
### 1.2 L10n
*** 
  
  L10n va définir la préférence linguistique de l'utilisateur en la stockant dans la variable I18n.locale. C'est donc ce qui  
  va permettre d'aiguiller I18n, en lui disant par exemple : si la locale vaut "fr", alors charge moi toutes les clefs 
  correspondantes pour me passer le site en français.
   
   
## 2. Mise en place basique
***


### 2.1 Installation :wrench:
***

On ne déroge pas à la règle, on commence par rajouter dans le Gemfile :
```
gem 'rails-i18n'
```
Puis évidemment, en console :
```
$ bundle install
```

### 2.2 Explications essentielles :books:
***

La méthode la plus importante à retenir de I18n est sûrement : 
```
translate "mot_clef_présent_dans_les_fichiers_yml" 
```
Celle-ci permet comme son l'indique de traduire un mot clef présent dans le fichier correspondant à la valeur stockée dans
I18n.locale. On la retrouvera alors dans les views, elle viendra remplacer les valeurs écrites habituellement "en dur" dans
les balises HTML.
Les fichiers de traduction sont regroupés dans config/locales; par défaut en.yml y est déjà présent. 

L'autre méthode similaire à retenir, moins utilisée mais qui peut s'avérer utile :
```
localize objet_time_ou_date
```
Celle-ci permet d'adapter le format de l'heure et de la date selon ce même paramètre locale.
On peut alors lui ajouter directement l'objet souhaité tel quel :
```
localize Time.now
```
On peut raccourcir ces deux méthodes simplement par "t" pour translate et "l" pour localize.

Ensuite, il y a quelques attribute readers et writers à connaître :

 - locale, qui permet d'obtenir / de règler la variable locale courante
 - default_locale, qui est règlée par défaut à :en
 - available_locales, qui permet d'instancier dans un array toutes les locales que l'on veut utiliser

### 2.3 Exemple pas à pas :runner:
***

Commençons par paramétrer nos différentes langues, disons qu'on veut un site adapté en français et en anglais.
On se retrouve dans config/locales et on crée un fichier :fr (:en est déjà là).

Puis on vient paramétrer la locale dans l'ApplicationController pour qu'il la mette bien à jour si jamais celle-ci change.
```
class ApplicationController < ActionController::Base
  before_action :set_locale
  
  private
  
  def set_locale
    if params[:locale]
      I18n.locale = params[:locale]
    end
  end
end
```
Ensuite on peut restreindre nos locales (pas obligatoire, mais indispensable pour l'exemple dans la 3e partie)
=> dans config/application.rb, dans la classe Application du module :
```
config.i18n.available_locales: [ :fr, :en ]
```
! Il faudra redémarrer le serveur pour prendre en compte les changements effectués dans ce fichier !

Bien, maintenant rajoutons deux boutons dans notre view pour permettre de changer la locale depuis le site :
```
<%= link_to_unless_current "fr", locale: :fr %>
<%= link_to_unless_current "en", locale: :en %>
```

Enfin, définissons dans notre view l'élément qui sera traduit, en omettant les commentaires sur l'originalité...
```
<%= t "hello" %>
```
"hello" constitue donc la clef que l'on va retrouver dans chaque fichier .yml

Plaçons la dans ceux-ci avec le format suivant :
```
fr: 
  hello: "Bonjour, monde ?"
``` 
qui correspondra alors avec en.yml :
```
en: 
  hello: "Hello, world ?"
``` 
Petite parenthèse : on peut imbriquer des mots clefs comme suit :
```
en: 
  h1:
    hello: "Hello, world ?"
``` 
Puis dans la view l'appeler en reliant chaque élément parent à son enfant par un point :
```
<%= t "h1.hello" %>
```

Et voilà ! Plus qu'à tester en local :D

Dernier point : /?locale=fr, faut avouer que c'est pas très sexy dans l'URL.
On peut résoudre ce détail dans config/routes.rb :
```
scope "(:locale)", locale: /#{I18n.available_locales.join("|")}/ do
  toutes_les_routes_que_vous_voulez 
end
```
Ce qui nous donne un URL avec un joli /fr/ :)


## 3. Détection des préférences de l'utilisateur :heart:
***


Ok, changer de langue avec un bouton, c'est cool.
Mais tomber sur sa propre langue dès l'arrivée sur le site, c'est encore mieux.

Les navigateurs gardent ces préférences dans une variable nommée ['HTTP_ACCEPT_LANGUAGE'] ;
Si on l'affiche dans la view on obtient quelquechose comme :
```
fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3
```
On y retrouve les langues que le client est capable d'interpréter.
La 1e langue, ici fr, représente la langue préférée du client. 
Elle est suivie de fr-FR, qui représente la localisation => ici le français de France (et non du Canada par exemple)
Enfin, on trouve dans "q=" un système de pondération entre 0 et 1 qui évalue la langue préférée.

Pour vous faciliter la tâche et faire le travail proprement, je vous recommande l'utilisation de la gem "http_accept_languages".
Vous connaissez le mot d'ordre pour l'installation...

Tout ce que vous avez alors à faire pour mettre la langue préférée de votre visiteur automatiquement, c'est de modifier le fichier ApplicationController que l'on a déjà modifié tout à l'heure :
```
class ApplicationController < ActionController::Base
  before_action :set_locale

  private

  def set_locale
    if !params[:locale]
      I18n.locale = http_accept_language.compatible_language_from(I18n.available_locales)
    else
      I18n.locale = params[:locale]
    end
  end
end
```
On rajoute simplement une condition dans laquelle on va rentrer dès l'arrivée sur le site, c'est à dire quand l'utilisateur n'a pas encore choisi de langue. Dans cette condition on instancie la valeur de la locale à la langue préférée si elle est disponible dans nos fichiers .yml.
Sinon, on va garder la locale que l'utilisateur aura choisi.

Enfin, vous pouvez ajouter à la view la valeur de la locale en cours, c'est toujours plus sympa de savoir où on se situe !
```
<%=  I18n.locale %>
```

Et voilà ! 
Ah oui, évidemment, j'ai oublié de préciser que vous allez avoir besoin d'un traducteur ! Sorry for this !
A part ça, vous n'avez plus d'excuses pour ne pas traduire vos sites favoris et ainsi mieux envahir le monde !
Je décline toutefois toute part de responsabilité dans cette dernière entreprise.

:pray: Namaste :pray:


## 4.Aller plus loin :rocket:
***


[Repo Github de i18n](https://github.com/ruby-i18n/i18n)

[Repo Github de http_accept_languages](https://github.com/iain/http_accept_languageSources )

[La doc officielle de la gem i18n](https://guides.rubyonrails.org/i18n.html)

[Vidéo la plus complète que j'ai pu trouver, il met en application différentes méthodes et exemples](https://www.youtube.com/watch?v=0CxTZRZ93II)

