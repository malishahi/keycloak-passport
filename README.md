# Keycloak Passport Strategy - oAuth2/OIDC

This library offers a production-ready and maintained Keycloak Passport connector that offers the following key features:

- Use multiple realms in the same application (multi-tenancy)

- Use with oAuth2/Open ID Connect 'clients' in keycloak

- Fetch users' data from keycloak automatically via the JSON API

## Listed Keycloak Extension

Check it out on [keycloak.org](https://www.keycloak.org/extensions.html)

## Why? Hasn't this already been done?

To a certain extent, yes. There are about 3 to 4 repos that brand themselves as 'Keycloak Passport', yet not a single one of them is actively maintained and most of them are either completely empty, don't allow using multiple realms, only implement part of the protocol, and/or don't fetch the user's data from Keycloak. There also exists a dedicated NodeJS connector by the Keycloak project itself, however, it is unusable if you are seeking to have Keycloak as 'yet another' passport strategy in your app. This project fills that gap.

## Usage

## Install

```bash
npm install @exlinc/keycloak-passport
```

### Import

```javascript
import KeycloakStrategy from "@exlinc/keycloak-passport";
```

### Initialize

```javascript
const AUTH_KEYCLOAK_CALLBACK = '/api/auth/keycloak/return';
const kcUri = `${process.env.KEYCLOAK_HOST}/auth/realms/${process.env.KEYCLOAK_REALM}/protocol/openid-connect`;

// Register the strategy with passport
passport.use(
  "keycloak",
  new KeycloakStrategy(
    {
      host: process.env.KEYCLOAK_HOST,
      realm: process.env.KEYCLOAK_REALM,
      clientID: process.env.KEYCLOAK_CLIENT_ID,
      clientSecret: process.env.KEYCLOAK_CLIENT_SECRET,
      callbackURL: `${AUTH_KEYCLOAK_CALLBACK}`,
      authorizationURL: `${kcUri}/auth`,
      tokenURL: `${kcUri}/token`,
      userInfoURL: `${kcUri}/userinfo`
    },
    (accessToken, refreshToken, profile, done) => {
      // This is called after a successful authentication has been completed
      // Here's a sample of what you can then do, i.e., write the user to your DB
      User.findOrCreate({ email: profile.email }, (err, user) => {
        assert.ifError(err);
        user.keycloakId = profile.keycloakId;
        user.imageUrl = profile.avatar;
        user.name = profile.name;
        user.save((err, savedUser) => done(err, savedUser));
      });
    }
  )
);
```

### Routes

```javascript
app.get('/api/auth/keycloak/',
    passport.authenticate("keycloak", (err, user) => {
        if (err || !user) {
            console.log(err, user);
            res.json({ err, user });
            return next();
        }
    })
);

// req.body.username is undefined here, since here keycloak redirects to callback url on our website, and info is {}
app.get('/api/auth/keycloak/return', (req, res, next) => {
    passport.authenticate("keycloak", (err, user, info) => {
        if (err || !user) {
            res.json({ err, user });
            return next();
        }

        req.logIn(user, err => {
            if (err)
                return next(err);

            //redirect to home page
            res.redirect('/api');
            // or just show a json status report.
            // return res.json({
            //     message: 'req.logIn execution was successful in retrun uri',
            //     user,
            //     fullname: user.fullName,
            //     email: user.email
            // });
        });
    })(req, res, next);
});

app.get('/api',
    (req, res) => {
        if (req.isAuthenticated()) {
            return res.json({ message: 'AUTH OK /api/', user: req.user });
        }

        return res.json({ message: 'NOT AUTH' });
    });
```

## Compatability with next-auth

There are some known issues with using this passportjs strategy with the latest versions of next-auth. Follow the discussion [here](https://github.com/exlinc/keycloak-passport/issues/1).

## Contributing/feedback

All forms of contribution are welcome via Issues and Pull-requests to this repo
