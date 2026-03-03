# 8. Programación de Interfaz Pública 

## 8.1 Configurar CORS en el backend
Antes de hacer peticiones a las APIs del backend se debe configurar el archivo **config/cors.php**, para que permita peticiones del dominio del proyecto frontend, de la siguiente manera:

```php
'paths' => ['api/*'],
    'allowed_origins' => ['http://localhost:5173'],
```
En Laravel 12, el archivo config/cors.php puede no aparecer por defecto. Para solucionarlo, publica la configuración ejecutando
```php
php artisan config:publish cors
```
