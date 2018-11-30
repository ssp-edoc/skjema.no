# Skjema.no 

Pakken inneholder koden for vise skjema.


## Installasjon.

Legg inn viewer.min.js nederst på HTML siden før body slutt tag.
```html
    <body>
    ....
    ....
        <script src="viewer.min.js"></script>
    </body>
```

Legg inn styles.css i head taggen.

```html
    <head>
        ...
        ...
        <link rel='stylesheet' href='styles.css' type='text/css'  />
    </head>
```

Opprett en tom element hvor å rendre innholdet av skjemet.

```html
<body>
    <div id="skjema"></div>
</body>

```

Start scriptet med kommandoen:

```javascript
document.addEventListener("DOMContentLoaded", function(){
    viewer
        .init({
            apiUrl: "https://api.lab.skjema.no/",
            customerId: "Din id her",
            autosaveIntervalSeconds: -1,
            hide: {
                title: true,
                buttons: true,
                summary: true,
            }
        })
        .session({
            singleLogOut: false,
            idleTimeoutMinutes: 60,
            warningTimeoutMinutes: 5,
        })
        .form({
            divId: "skjema",
            refId: function() {
                var id = detailsData.Instance.RefId;
    
                if(detailsData.Case.DefaultDiff) {
                    id += "-" + detailsData.Case.DefaultDiff;
                }
    
                return  id;
            },
            onSaving: function (proceed) {
                proceed();
            },
        })
        .catch(function (error) {
            alert(error);
        })
        .finally(function () {
            // Legg til callback funksjon ved behov
        });
});

```

## API

### init:
Starter opp skjema. 
```javascript
viewer.init({...})
```

#### init object: 

```javascript
viewer.init({
    apiUrl : "url her", // Forklaring
    customerId: "din id her", // Forklaring
    autosaveIntervalSeconds: -1, // Forklaring
    hide : {
        title:true, // Forklaring
        buttons:true, // Forklaring
        summary : true // Forklaring
    }
})
```

### session:
Forklar session her
```javascript
viewer
    .init({...})
    .session({})
```

#### session object:
```javascript
viewer
    .init({...})
    .session({
        singleLogOut: false, // Forklaring her
        idleTimeoutMinutes: 60, // Forklaring her
        warningTimeoutMinutes: 5, // Forklaring her
    })
```

### form:

Forklar session her
```javascript
viewer
    .init({...})
    .session({...})
    .form({})
```

#### form object:

Forklar session her
```javascript
viewer
    .init({...})
    .session({...})
    .form({
        divId: "bodycontent", // Forklaring her
        refId: function() {
            // Forklaring her
        },
        onSaving: function (proceed) {
            // Forklaring her
        },
    })
```

## Innhold

### Viewer

- Babel Polyfill
- Handlebars
- jQuery
- jQuery UI
- DropZone
- i18next
- mathjs
- moment.js
- opn
- urijs 

### Styles

- Bootstrap 3.3.7
- Budicons
