# webpack-howto

## Στόχος για αυτόν τον οδηγό

Αυτό είναι ένα σύνολο συμβουλών για το πως να κάνετε πράγματα με το webpack. Επίσης περιλαμβάνει μόνο πράγματα που χρησιμοποιούμε στο Instagram.

Η συμβουλή μου: ξεκινείστε με αυτό ως webpack docs και μετά κοιτάξτε τα επίσημα docs.

## Προαπαιτούμενα

  * Ξέρετε browserify, RequireJS ή κάτι παρόμοιο
  * Βλέπετε τα θετικά των:
    * Bundle splitting
    * Async loading
    * Packaging static assets όπως εικόνες και CSS

## 1. Γιατί webpack?


  * **Είναι σαν το browserify**, αλλά μπορεί να χωρίσει την εφαρμογή σας σε πολλαπλά αρχεία. Αν έχετε πολλαπλές σελίδες σε μια μονοσέλιδη εφαρμογή, ο χρήστης κατεβάζει κώδικα μόνο για αυτή τη σελίδα. Αν πάνε σε άλλη σελίδα, δεν ξανακατεβάζουν τον κοινό κώδικα.

  * **Συχνά αντικαθιστά το grunt ή το gulp** γιατί μπορεί να χτίσει και να δεσμεύσει CSS, preprocessed CSS, compile-to-JS γλώσσες και εικόνες μέσα σε άλλα πράγματα.

Υποστηρίζει AMD και CommonJS, όπως και άλλα module systems (Angular, ES6). Αν δεν ξέρετε τι να χρησιμοποιήσετε, χρησιμοποιήστε CommonJS.

## 2. Webpack για ανθρώπους με γνώσεις στο Browserify

Αυτά είναι αντίστοιχα:

```js
browserify main.js > bundle.js
```

```js
webpack main.js bundle.js
```

Ωστόσο, το webpack είναι πιο δυνατό από το Browserify, οπότε γενικά θέλετε να κάνετε ένα `webpack.config.js` για να έχετε τα πράγματά σας οργανωμένα:

```js
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'       
  }
};
```

Αυτό είναι απλά JS, οπότε μπορείτε να βάλετε κανονικό κώδικα εκεί μέσα.

## 3. Πως να χρησιμοποιείτε το webpack

Πηγαίντε στο directory που περιέχει το `webpack.config.js` και τρέξτε:

  * `webpack` για να χτίζετε μια φορά για τον προγραμματισμό
  * `webpack -p` για να χτίζετε μια φορά για παραγωγή (minification)
  * `webpack --watch` για συνεχή σταδιακή ανάπτυξη (γρήγορο!)
  * `webpack -d` για να περιλαμβάνετε και τα source maps

## 4. Compile-to-JS γλώσσες

Τα αντίστοιχα transforms του browserify και τα plugins για το RequireJS είναι τα **loader**. Ορίστε πως μπορείτε να μάθετε στο webpack πως να φορτώνει CoffeeScript και το JSX+ES6 του Facebook (πρέπει να τρέξετε `npm install babel-loader coffee-loader`):

Κοιτάξτε επίσης το [babel-loader installation instructions](https://www.npmjs.com/package/babel-loader) για επιπλέον dependencies (tl;dr τρέξτε `npm install babel-core babel-preset-es2015 babel-preset-react`).

```js
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'       
  },
  module: {
    loaders: [
      { test: /\.coffee$/, loader: 'coffee-loader' },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        query: {
          presets: ['es2015', 'react']
        }
      }
    ]
  }
};
```

Για να ενεργοποιήσετε απαιτούμενα αρχεία χωρίς να προσδιορίζετε το extension, πρέπει να προσθέσετε μια `resolve.extensions` παράμετρο προσδιορίζοντας ποια αρχεία να ψάξει το webpack:

```js
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'       
  },
  module: {
    loaders: [
      { test: /\.coffee$/, loader: 'coffee-loader' },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        query: {
          presets: ['es2015', 'react']
        }
      }
    ]
  },
  resolve: {
    // μπορείτε πλεον να κάνετε require('file') αντί για require('file.coffee')
    extensions: ['', '.js', '.json', '.coffee']
  }
};
```


## 5. Stylesheets και εικόνες

Πρώτα ανανεώστε τον κώδικά σας για να κάνετε `require()` τα στατικά assets (ονομασμένα όπως θα ήταν με το `require()` του node):

```js
require('./bootstrap.css');
require('./myapp.less');

var img = document.createElement('img');
img.src = require('./glyph.png');
```

Όταν κάνετε require CSS (ή less, κλπ.), το webpack κάνει inline το CSS ως string μέσα στο JS bundle και το `require()` θα εισάγει ένα `<style>` tag στην σελίδα. Όταν κάνετε require εικόνες, το webpack κάνει inline ένα URL στην εικόνα μέσα στο bundle και το επιστρέφει απο το `require()`.

Αλλά πρέπει να μάθετε το webpack να το κάνει αυτό (ξανά, με loaders):

```js
// webpack.config.js
module.exports = {
  entry: './main.js',
  output: {
    path: './build', // Εδώ θα πάνε οι εικόνες ΚΑΙ η js
    publicPath: 'http://mycdn.com/', // Αυτό χρησιμοποιείται για να δημιουργεί URLs σε π.χ. εικόνες
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      { test: /\.less$/, loader: 'style-loader!css-loader!less-loader' }, // χρησιμοποιείτε ! για να κάνετε chain τους loaders
      { test: /\.css$/, loader: 'style-loader!css-loader' },
      { test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192' } // inline base64 URLs για <=8k εικόνες, κατ' ευθείαν URLs για τα υπόλοιπα
    ]
  }
};
```

## 6. Feature flags

Έχουμε κώδικα που θέλουμε να κάνουμε gate μόνο στο dev περιβάλλον μας (όπως το logging) και τα εσωτερικά dogfooding servers (όπως unreleased features που τεστάρουμε). Στον κώδικά σας αναφέρονται ως magic globals:

```js
if (__DEV__) {
  console.warn('Extra logging');
}
// ...
if (__PRERELEASE__) {
  showSecretFeature();
}
```

Μετά μάθετε το webpack αυτά τα magic globals:

```js
// webpack.config.js

// definePlugin παίρνει strings και τα βάζει, οπότε μπορείτε να βάλετε JS strings αν θέλετε.
var definePlugin = new webpack.DefinePlugin({
  __DEV__: JSON.stringify(JSON.parse(process.env.BUILD_DEV || 'true')),
  __PRERELEASE__: JSON.stringify(JSON.parse(process.env.BUILD_PRERELEASE || 'false'))
});

module.exports = {
  entry: './main.js',
  output: {
    filename: 'bundle.js'       
  },
  plugins: [definePlugin]
};
```

Μετά μπορείτε να χτίσετε με `BUILD_DEV=1 BUILD_PRERELEASE=1 webpack` από το console. Σημειώστε πως αφού το `webpack -p` τρέχει το uglify, οτιδήποτε μέσα σε κάποιο από αυτά τα blocks θα το πετάξει έξω, οπότε δεν θα φανερωθούν μυστικά features ή strings.

## 7. Πολλαπλά entrypoints

Ας πούμε πως έχετε μια σελίδα profile και μια news feed. Δεν θέλετε ο χρήστης να κατεβάζει τον κώδικα για το feed αν χρησιμοποιούν απλά το profile. Οπότε ας δημιουργήσουμε πολλαπλά bundles: ένα "main module" (ονομαζόμενο entrypoint) ανά σελίδα:

```js
// webpack.config.js
module.exports = {
  entry: {
    Profile: './profile.js',
    Feed: './feed.js'
  },
  output: {
    path: 'build',
    filename: '[name].js' // Template βασισμένο σε keys στο παραπάνω entry
  }
};
```

Για το profile, εισάγετε `<script src="build/Profile.js"></script>` στην σελίδα σας. Κάντε παρόμοια για το feed.

## 8. Βελτιστωποίηση κοινού κώδικα

Το feed και το profile μοιράζονται πολλά (όπως το React και τα κοινά stylesheets και components). Το webpack μπορεί να καταλάβει ποια κοινά έχουν και να δημιουργήσει ένα κοινό bundle που μπορεί να γίνει cached μεταξύ σελίδων:

```js
// webpack.config.js

var webpack = require('webpack');

var commonsPlugin =
  new webpack.optimize.CommonsChunkPlugin('common.js');

module.exports = {
  entry: {
    Profile: './profile.js',
    Feed: './feed.js'
  },
  output: {
    path: 'build',
    filename: '[name].js' // Template βασισμένο σε keys στο παραπάνω entry
  },
  plugins: [commonsPlugin]
};
```

Προσθέστε `<script src="build/common.js"></script>` πριν το script tag που προσθέσατε στο προηγούμενο βήμα και απολαύστε τσάμπα caching.

## 9. Async loading

CommonJS είναι synchronous αλλά το webpack μας επιτρέπει έναν τρόπο να προσδιορίσουμε dependencies asynchronously. Αυτό είναι χρήσιμο για client-side routers, όπου θέλετε ένα router σε κάθε σελίδα, αλλά δεν θέλετε να πρέπει να κατεβάσετε features πριν τα χρειαστείτε.

Προσδιορίστε το **split point** όπου θέλετε να φορτώσετε asynchronously. Για παράδειγμα:

```js
if (window.location.pathname === '/feed') {
  showLoadingState();
  require.ensure([], function() { // αυτό το συντακτικό είναι περίεργο αλλά λειτουργεί
    hideLoadingState();
    require('./feed').show(); // όταν αυτό το function καλείται, το module είναι σίγουρο ότι θα είναι synchronously διαθέσιμο.
  });
} else if (window.location.pathname === '/profile') {
  showLoadingState();
  require.ensure([], function() {
    hideLoadingState();
    require('./profile').show();
  });
}
```

Το webpack θα κάνει τα υπόλοιπα και θα δημιουργήσει έξτρα **chunk** αρχεία και θα τα φορτώσει για σας.

Το webpack θα θεωρήσει ότι αυτά τα αρχεία είναι στο root directory όταν τα φορτώνετε σε ένα html script tag για παράδειγμα. Μπορείτε να χρησιμοποιείτε `output.publicPath` για να το διαμορφώσετε αυτό.

```js
// webpack.config.js
output: {
    path: "/home/proj/public/assets", //path στο οποίο το webpack θα χτίσει τα πράγματά σας
    publicPath: "/assets/" //path που θα θεωρηθεί όταν κάνετε require τα αρχεία σας
}
```

## Επιπλέον resources

Ρίξτε μια ματιά σε ένα πραγματικό παράδειγμα για να δείτε πως μια πετυχιμένη ομάδα χρησιμοποιεί το webpack: http://youtu.be/VkTCL6Nqm6Y
Αυτό είναι ο Pete Hunt στο OSCon που μιλάει για το webpack στο Instagram.com

## FAQ

### webpack δεν φαίνεται modular

Το webpack είναι **απίστευτα** modular. Αυτό που κάνει το webpack σπουδαίο είναι πως αφήνει plugins να εκτελούνται σε πιο πολλά μέρη κατά τη διάρκεια του build process, συγκρινόμενο με άλλες εναλλακτικές όπως το browserify και requirejs. Πολλά πράγματα που φαίνονται χτισμένα στο core είναι απλά plugins που φορτώνονται από προεπιλογή και μπορούν να γίνουν overridden (π.χ. το CommonJS require() parser).
