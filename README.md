Emoji is a RESTful API example written in the Nature programming language. With the help of Nature's package management tool, emoji-api can also be used as a web framework.

## Installation

Add dependency to package.toml

```toml
[dependencies]
emoji = { type = "git", version = "v0.1.0", url = "github.com/weiwenhao/emoji-api" }
```

## Quickstart

```js
import json
import fmt

import emoji.services.api

fn main() {
    var app = api.new()
    app.use(fn(ptr<api.ctx_t> c):void! { 
        fmt.printf('request log %d %s\n', c.req.method, c.req.url)
        c.next()
    })

    app.get('/hello/:msg', fn(ptr<api.ctx_t> c):void! {
        return c.json(200, {
            'message': 'ok',
            'data': 'hello ' + c.param('msg'),
        })
    })

    app.listen(8888)
}
```


## Module Examples

### DB

```js
import emoji.services.db

// dep: db = { type = "git", version = "v0.13.0", url = "github.com/weiwenhao/dbdriver" }
import db.result as db_result

fn main() {
    // mysql://user:password@host:port/database
    db.init('mysql://root:root@127.0.0.1:3306/emoji')

    // get conn from pool
    var result = db.conn().query('SELECT * FROM users')

    type user_t = struct{
        int id
        string nickname 
        string? email
    }

    var list = db_result.scan<user_t>(result)
    for u in list {
        println(u.id, u.nickname, u.email)
    }
}
```


### JWT Token

```js
import emoji.services.jwt

fn main() {
    jwt.init('Secret Key', 7200, 'emoji')

   // Generate token
    {string:any} custom_data = {
        'username': 'john_doe',
        'email': 'john@example.com',
    }
    
    var token = jwt.generate('user123', custom_data, 0)
    println('Generated token:', token)

    // Verify token
    var jwt_token = jwt.verify(token) catch e {
        throw errorf('verify failed: %s', e.msg())
    }

    println('User ID (sub):', jwt_token.payload.sub)
}
```

### Auth middleware

```js
import json
import fmt
import emoji.services.api
import emoji.middlewares.auth

fn main() {
    var app = api.new()

    // need Header Authorization: Bearer Token
    app.get('/users/auth', auth.handle, fn(ptr<api.ctx_t> c):void! {
        var user_id = c.get('auth_id') as int
        // ...
    })
}
```

### Config

```json
{
    "db_host": "127.0.0.1:3306",
    "db_database": "emoji",
    "db_username": "username",
    "db_password": "password",

    "mail_host": "smtp.gmail.com",
    "mail_port": 465,
    "mail_username": "username@gamil.com",
    "mail_password": "password",

    "jwt_secret": "jwt.secret"
}

```

Import and load config

```js
import json
import fmt
import emoji.services.config

fn main() {
    var c = config.load('./config.json')
    println(c.db_host, c.db_database, c.db_username, c.db_password)
}
```


### Mail

Only supports SSL mail

```js
import emoji.services.mail

fn main() {
    // SMTP configuration
    var config = mail.config_t{
        host = 'smtp.163.com',
        port = 465,
        username = 'xxxxxx',
        password = 'xxxxxx',
        helo_domain = 'localhost',
    }

    // Create plain text mail
    var m = mail.mail_t{
        from = 'username@163.com',
        to = ['username@qq.com'],
        cc = [],
        bcc = [],
        subject = 'Test Mail - Nature SMTP Client',
        body = 'This is a test mail plain text content.\n\nSent time: 2025',
        headers = {
            'X-Mailer': 'Nature SMTP Client v1.0',
        },
    }

    // Send mail
    var client = mail.client_new(config)
    client.send(m) catch e {
        println('Mail sending failed:', e.msg())
    }

    client.close()    
}
```

## License
MIT