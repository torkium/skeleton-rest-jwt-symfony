Openssl doit être installé !

1 - Créer le projet symfony

    composer create-project symfony/website-skeleton symfony_rest_auth_jwt

2 - Installer les packages nécessaires

    composer require jms/serializer-bundle
    composer require friendsofsymfony/rest-bundle
    composer require symfony/maker-bundle    
    composer require symfony/orm-pack --with-all-dependencies
    composer require lexik/jwt-authentication-bundle:*

2 - Modifier le fichier .env pour la configuration de la base de données

    DATABASE_URL="mysql://root:@127.0.0.1:3307/symfony_rest_jwt?serverVersion=10.10.2-MariaDB"

3 - Créer la base de données 

    php bin/console doctrine:database:create

4  Configurer fos_rest. Dans config/packages/fos_rest.yaml

    fos_rest:
        format_listener:
            rules:
                - { path: ^/api, prefer_extension: true, fallback_format: json, priorities: [ json, html ] }

5 - Créer une entité User

    php bin/console make:user

6 - Ajouter un champs username à l’entité User

    #[ORM\Column(length: 180, unique: true)]
    private ?string $username = null;

    public function getUsername(): string
    {
        return $this->username;
    }
  
    public function setUsername(string $username): self
    {
        $this->username = $username;
  
        return $this;
    }


7 - Créer la migration et l’appliquer

    php bin/console make:migration
    php bin/console doctrine:migrations:migrate


8 - Configurer les clés public et private de jwt

    php bin/console lexik:jwt:generate-keypair

9 - Mettre à jour le fichier de config des routes pour ajouter une route de login check

    api_login_check:
        path: /api/login_check

10 - Créer un hash de password

    php bin/console security:hash-password 

11 - Créer l’utilisateur en base de données avec le hash de password, et [] dans le rôle.

    INSERT INTO `user` (`id`, `email`, `username`, `roles`, `password`) VALUES (NULL, 'julien.taront@gmail.com', 'torkium', '[]', '$2y$13$HUbtMvkr9j4WqgFJxIHv.O5X1/otD1MwCphYyKB.YU0iHCix4/U/q');

12 - Configurer le Secyrity.yaml

    security:
        enable_authenticator_manager: true
        password_hashers:
            App\Entity\User: 'auto'
            Symfony\Component\Security\Core\User\PasswordAuthenticatedUserInterface:
                algorithm: 'auto'
                cost:      15
        providers:
            app_user_provider:
                entity:
                    class: App\Entity\User
                    property: username
        firewalls:
            login:
                pattern: ^/api/login
                stateless: true
                json_login:
                    check_path: /api/login_check
                    success_handler: lexik_jwt_authentication.handler.authentication_success
                    failure_handler: lexik_jwt_authentication.handler.authentication_failure
    
            api:
                pattern:   ^/api
                stateless: true
                jwt: ~
            dev:
                pattern: ^/(_(profiler|wdt)|css|images|js)/
                security: false
            main:
                lazy: true
                provider: app_user_provider
      
        access_control:
            - { path: ^/api/register, roles: PUBLIC_ACCESS  }
            - { path: ^/api/login, roles: PUBLIC_ACCESS  }
            - { path: ^/api,       roles: IS_AUTHENTICATED_FULLY }

13 - Tester l'API

Via postman, créer une requête GET

    http://127.0.0.1:8000/api/login_check
Avec un body de type json

    {
        "username": "torkium",
        "password": "password"
    }
