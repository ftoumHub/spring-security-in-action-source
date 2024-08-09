## 2.3 Overriding default configurations

Maintenant que vous connaissez les configurations par défaut de votre premier projet, il est temps de voir comment vous pouvez les remplacer. Il est important de comprendre les options dont vous disposez pour substituer les composants par défaut, car c'est ainsi que vous pouvez intégrer vos propres implémentations personnalisées et appliquer la sécurité de manière adaptée à votre application. Comme vous l'apprendrez dans cette section, le processus de développement concerne également la façon dont vous écrivez les configurations pour que vos applications restent hautement maintenables.

Avec les projets sur lesquels nous allons travailler, vous trouverez souvent plusieurs façons de remplacer une configuration. Cette flexibilité peut créer de la confusion. Je vois fréquemment un mélange de styles différents pour configurer diverses parties de Spring Security dans la même application, ce qui est indésirable. Ainsi, cette flexibilité s'accompagne d'une mise en garde. Vous devez apprendre à choisir parmi ces options, donc cette section concerne également la connaissance de vos options.

Dans certains cas, les développeurs choisissent d'utiliser des beans dans le contexte Spring pour la configuration. Dans d'autres cas, ils remplacent diverses méthodes pour le même objectif. La rapidité avec laquelle l'écosystème Spring a évolué est probablement l'un des principaux facteurs ayant généré ces multiples approches. Configurer un projet avec un mélange de styles n'est pas souhaitable, car cela rend le code difficile à comprendre et affecte la maintenabilité de l'application. Connaître vos options et savoir comment les utiliser est une compétence précieuse, qui vous aide à mieux comprendre comment configurer la sécurité au niveau de l'application dans un projet.

Dans cette section, vous apprendrez comment configurer un **UserDetailsService** et un **PasswordEncoder**. Ces deux composants interviennent généralement dans l'authentification et la plupart des applications les personnalisent en fonction de leurs besoins. Bien que nous discutions des détails sur leur personnalisation dans les chapitres 3 et 4, il est essentiel de voir comment intégrer une implémentation personnalisée. Les implémentations que nous utilisons dans ce chapitre sont toutes fournies par Spring Security.



### 2.3.1 Customizing user details management

````shell
curl -u john:12345 http://localhost:8080/hello
````