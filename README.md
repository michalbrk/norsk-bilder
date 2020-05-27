# norsk_bilder
This is a showcase project build mainly for the portfolio purposes
and to be the point of reference for employers checking the skills level.

The project provides a simple search engine for the Norwegian-related pictures.
For the API provider I am using Pixabay here, every query is being checked for
the relation to Norway, based on the pre-defined list from within the project.

The stack here is:
- Vue
- Vuetify
- Axios

I am also using localStorage for caching requests and EventBus as the
components-communication engine.
## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
