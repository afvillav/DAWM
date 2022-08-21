---
theme: jekyll-theme-leap-day
---

[Regresar](/DAWM-2022/)

Express - Cookies y Sesión
==========================

Una cookie HTTP, cookie web o cookie de navegador es una pequeña pieza de datos que un servidor envía a el navegador web del usuario. El navegador guarda estos datos y los envía de regreso junto con la nueva petición al mismo servidor. Las cookies se usan generalmente para decirle al servidor que dos peticiones tienen su origen en el mismo navegador web lo que permite, por ejemplo, mantener la sesión de un usuario abierta. Las cookies permiten recordar la información de estado en vista a que el protocolo HTTP es un protocolo sin estado.

Las cookies se utilizan principalmente con tres propósitos:

* Gestión de Sesiones. Inicios de sesión, carritos de compras, puntajes de juegos o cualquier otra cosa que el servidor deba recordar
* Personalización. Preferencias de usuario, temas y otras configuraciones
* Rastreo. Guardar y analizar el comportamiento del usuario

Fuente: [MDN - HTTP cookies](https://developer.mozilla.org/es/docs/Web/HTTP/Cookies)

Proyecto en Express
===================

* * *

* Utiliza el proyecto que desarrollaste con los tutoriales de [Express - Bases](https://dawfiec.github.io/DAWM-2022/tutoriales/express_bases.html), [Express - Bootstrap](https://dawfiec.github.io/DAWM-2022/tutoriales/express_bootstrap.html), [Express - Formularios](https://dawfiec.github.io/DAWM-2022/tutoriales/express_forms.html), [Express - Layouts y Partials](https://dawfiec.github.io/DAWM-2022/tutoriales/express_partials.html), [Express - ORM (Básico)](https://dawfiec.github.io/DAWM-2022/tutoriales/express_ormbasico.html), [Express - Parámetros de consulta y Parámetros de ruta](https://dawfiec.github.io/DAWM-2022/tutoriales/express_pcpr.html) y [Express - REST](https://dawfiec.github.io/DAWM-2022/tutoriales/express_rest.html).

* Instala las dependencias, con: `npm install`
* Verifica que funcione correctamente al levantar los servicios: `SET DEBUG=misitio:\* & npm start`
* Verifique el acceso libre a todas las URLs:
  + Principal: `http://localhost:3000/`
  + Productos: `http://localhost:3000/productos`
  + Reporte: `http://localhost:3000/reporte`
  + Login: `http://localhost:3000/login`


Autorización
============

* * *

* Instale [**express-session**](https://www.npmjs.com/package/express-session) , con: `npm install express-session`

* Modifique `app.js`:
  + Agregue la referencia a **express-session**, con: 

    <pre><code>
    ...
    var cors = require('cors')
    <b style="color:red">
    var session = require('express-session');
    </b>
    var indexRouter = require('./routes/index');
    ...
    </code></pre>

  + Añada el _middleware_ session a la aplicación, con:

    <pre><code>
    ...
    var app = express();
    <b style="color:red">
    app.use(session({
        secret: '2C44-4D44-WppQ38S',
        resave: true,
        saveUninitialized: false,
        cookie: { maxAge: 30000 }
    }));
    </b>
    // view engine setup
    ...
    </code></pre>

* Agregue el **middleware** de _autorización_

  + Cree la carpeta `middlewares`
  + Agregue el _script_ de autorización en `middlewares\auth.js`:
  
    <pre><code>
    var express = require('express');
    var router = express.Router();

    let bd = {  
      'usuario': 'abc',  
      'contrasenia': '123'  
    }

    var auth = (req, res, next) => {
      if (req.session && req.session.user === bd['usuario'] && req.session.admin)
        return next();
      else
        return res.sendStatus(401);
    };

    module.exports = auth;
    </code></pre>

* Modifique `app.js`:
  + Agregue la referencia al **middleware**, con:  

    <pre><code>
    var apiRouter = require('./routes/api');
    <b style="color:red">
    var auth = require('./middlewares/auth');
    </b>
    var app = express();
    </code></pre>

  + Agregue el _middleware_ a la ruta raíz `/`

    <pre><code>
    ...
    app.use('/', auth, indexRouter);
    ...
    </code></pre>

  + Verifique el acceso **restringido** a todas las URLs:
    - Principal: `http://localhost:3000/`
    - Productos: `http://localhost:3000/productos`
    - Reporte: `http://localhost:3000/reporte`
    - Login: `http://localhost:3000/login`

* Autorice la ruta `/login`
  + En `app.js`, coloque la ruta `/login` antes de la ruta raíz `/`

    <pre><code>
    ...
    app.use('/login', loginRouter);
    app.use('/', auth, indexRouter);
    ...
    </code></pre>

  + Verifique el acceso **restringido** a todas las URLs:
    - Principal: `http://localhost:3000/`
    - Productos: `http://localhost:3000/productos`
    - Reporte: `http://localhost:3000/reporte`

  + Verifique el acceso **libre** a la URL:
    - Login: `http://localhost:3000/login`

Autenticación
============

* * *

* Modifique `routes/login.js`:
  + Agregue la instanciación de la sesión, con:

    <pre><code>
    ...
    if(usuario == bd['usuario'] && contrasenia == bd['contrasenia']) {
      <b style="color:red">
      req.session.user = bd['usuario'];
      req.session.admin = true;  
      </b>
      res.redirect('/');  
    } else {  
    ...
    </code></pre>

* Acceda a la ruta `/login`
  + Revise las **cookies de sesión** en el inspector del navegador

<p align="center">
  <img src="imagenes/nosession.png">
</p>

  + Ingrese las credenciales de usuario `abc` y contraseña `123`
  + Luego de la redirección, revise las **cookies de sesión**

<p align="center">
  <img src="imagenes/session.png">
</p>

* Cerrar sesión
  + Modifique el partial `views/partials/header.ejs`. Agregue la referencia a `/logout`

    ```
    ...
    <a class="nav-link px-3" href="/logout">Sign out</a>
    ...
    ```

  + Modifique el ruteador `routes/login.js`. Agregue el controlador para el método **GET** de la subruta `/out`

    <pre><code>
    ...
    router.get('/out', function(req, res, next) { 
      req.session.destroy();
      res.redirect('/login')
    });
    ...
    </code></pre> 

  + De clic en la opción **`Sign out`** de la esquina superior a la derecha.

Rastreo
=========

* * *

* Instale [**cookie-parser**](https://www.npmjs.com/package/cookie-parser) , con: `npm install --save cookie-parser`
* Modifique `app.js`:
  + Verifique o agregue la referencia a **cookie-parser**, con: 

    <pre><code>
    ...
    var path = require('path');
    <b style="color:red">
    var cookieParser = require('cookie-parser');
    </b>
    var logger = require('morgan');
    ...
    </code></pre>

  + Verifique o añada el _middleware_ session a la aplicación, con:

    <pre><code>
    ...
    app.use(express.urlencoded({ extended: false }));
    <b style="color:red">
    app.use(cookieParser());
    </b>
    ...
    </code></pre>

* Agregue el **middleware** de _rastreo_

  + En la carpeta `middlewares` agregue el _script_ de rastreo en `middlewares\tracing.js`:
  
    ```
    var express = require('express');
    var router = express.Router();

    var tracing = (req, res, next) => {
      res.cookie('tracing', req._parsedOriginalUrl.path , {expire : new Date() + 9999});
      return next();
    };
    module.exports = tracing;
    ```

* Modifique `app.js`:
  + Agregue la referencia al **middleware**, con:  

    <pre><code>
    var apiRouter = require('./routes/api');
    var auth = require('./middlewares/auth');
    <b style="color:red">
    var tracing = require('./middlewares/tracing');
    </b>
    var app = express();
    </code></pre>

  + Agregue el _middleware_ a la ruta raíz `/`

    <pre><code>
    ...
    app.use('/', auth, tracing, indexRouter);
    ...
    </code></pre>

* Modifique `routes/login.js`:
  + Agregue el rastreo para el redireccionamiento, con:

  <pre><code>
  if(usuario == bd['usuario'] && contrasenia == bd['contrasenia']) {
    req.session.user = bd['usuario'];
    req.session.admin = true;

    <b style="color:red">
    let tracing = req.cookies.tracing  || ''
    if(tracing.length > 0)
      res.redirect(tracing)   
    else
      res.redirect('/');
    </b> 
  } else {
  </code></pre>

* Después de hacer realizar una serie de rutas dentro de la aplicación, verifique el valor de la cookie **tracing**
  
  + Cierre y abra la sesión para comprobar el redireccionamiento.

<p align="center">
  <img src="imagenes/tracing.png">
</p>



Referencias 
===========

* * *

* HTTP cookies - HTTP MDN. (2022). Retrieved 21 August 2022, from https://developer.mozilla.org/es/docs/Web/HTTP/Cookies
* Manejo de Cookies en Express.js · GitBook. (2021). Retrieved 23 August 2021, from https://ull-esit-pl-1617.github.io/estudiar-cookies-y-sessions-en-expressjs-victor-pamela-jesus/cookies/chapter5.html 
* Sessions en ExpressJS · GitBook. (2021). Retrieved 23 August 2021, from https://ull-esit-dsi-1617.github.io/estudiar-cookies-y-sessions-en-expressjs-alejandro-raul-35l2-p4/sessionsexpress.html