# Créer une nouvelle version d'Electron

Ce document décrit le processus pour créer une nouvelle version d’Electron.

## Créer une branche temporaire

Créez une nouvelle branche depuis `master`. Nommez-la `release` ou comme vous le souhaitez.

Remarque : Si vous créez une version backport, vous devez utiliser `1-6-x`, `1-7-x`, etc... au lieu de `master`.

```sh
git checkout master
git pull
git checkout -b release
```

Cette branche est créée comme une mesure de précaution pour empêcher tout PRs fusionnées de se faufiler dans un communiqué, entre le moment où la branche temporaire de nouvelle version est créée et les builds de CI sont terminés.

## Rechercher les drafts existants

Le script d'upload [regarde les versions draft existant](https://github.com/electron/electron/blob/7961a97d7ddbed657c6c867cc8426e02c236c077/script/upload.py#L173-L181). Pour éviter que votre nouvelle version écrase un draft existant, consultez [la page releases](https://github.com/electron/electron/releases) et assurez-vous qu'il n'y ai aucun drafts.

## Changer le numéro de version

Exécutez le script `bump-version`, et indiquez `major`, `minor` ou `patch` comme paramètre :

```sh
npm run bump-version -- patch
git push origin HEAD
```

Cela va changer le numéro de version dans plusieurs fichiers. Voir [ce changement de version](https://github.com/electron/electron/commit/78ec1b8f89b3886b856377a1756a51617bc33f5a) pour exemple.

La plupart des nouvelles versions seront de niveau `patch`. Les mises à jour de Chrome ou d'autres changements majeurs doivent utiliser le niveau `minor`. Pour plus d’informations, consultez [electron-versioning](/docs/tutorial/electron-versioning.md).

## Attendez la compilation :hourglass_flowing_sand:

La présence du mot [`Bump`](https://github.com/electron/electron/blob/7961a97d7ddbed657c6c867cc8426e02c236c077/script/cibuild-linux#L3-L6) dans le message du commit est réalisée par le script `bump-version` qui va [déclencher le processus de versionnage](https://github.com/electron/electron/blob/7961a97d7ddbed657c6c867cc8426e02c236c077/script/cibuild#L82-L96).

Pour surveiller la progression de la compilation, allez voir les pages suivantes :

- [208.52.191.140:8080/view/All/builds](http://208.52.191.140:8080/view/All/builds) pour Mac et Windows
- [jenkins.githubapp.com/label/chromium/](https://jenkins.githubapp.com/label/chromium/) pour Linux

## Notes de compilation

Writing release notes is a good way to keep yourself busy while the builds are running. For prior art, see existing releases on [the releases page](https://github.com/electron/electron/releases).

Astuces :

- Each listed item should reference a PR on electron/electron, not an issue, nor a PR from another repo like libcc.
- No need to use link markup when referencing PRs. Strings like `#123` will automatically be converted to links on github.com.
- To see the version of Chromium, V8, and Node in every version of Electron, visit [atom.io/download/electron/index.json](https://atom.io/download/electron/index.json).

### Versions patch

Pour une version `patch`, utilisez le format suivant :

    ## Bug Fixes
    
    * Fixed a cross-platform thing. #123
    
    ### Linux
    
    * Fixed a Linux thing. #123
    
    ### macOS
    
    * Fixed a macOS thing. #123
    
    ### Windows
    
    * Fixed a Windows thing. #1234
    
    ## API Changes
    
    * Changed a thing. #123
    
    ### Linux
    
    * Changed a Linux thing. #123
    
    ### macOS
    
    * Changed a macOS thing. #123
    
    ### Windows
    
    * Changed a Windows thing. #123
    

### Versions mineures

Pour une version `minor` (qui est normalement une mise à jour de Chromium, et potentiellement aussi une mise à jour de Node), par exemple la `1.8.0`, utilisez ce format :

    **Note:** This is a beta release. This is the first release running on upgraded versions of Chrome/Node.js/V8 and most likely will have have some instability and/or regressions.
    
    Please file new issues for any bugs you find in it.
    
    This release is published to [npm](https://www.npmjs.com/package/electron) under the `beta` tag and can be installed via `npm install electron@beta`.
    
    ## Upgrades
    
    - Upgraded from Chrome `oldVersion` to `newVersion`. #123
    - Upgraded from Node `oldVersion` to `newVersion`. #123
    - Upgraded from v8 `oldVersion` to `newVersion`. #9116
    
    ## Other Changes
    
    - Some other change. #123
    

## Modifier la version draft

1. Visitez [la page de parutions](https://github.com/electron/electron/releases) et vous verrez un nouveau projet de version avec les notes de publication.
2. Modifiez la version et ajouter des notes de publication.
3. Assurez-vous que la case à cocher `prerelease` est cochée. Elle est cochée par défaut pour les version d'Electron >=1.7
4. Cliquez sur 'Save draft'. **Ne cliquez pas sur 'Publish release' !**
5. Attendez que toutes les compilations sont passées avant de continuer. 

## Fusionner une branche temporaire

Fusionner la branche temporaire dans master, sans créer un merge commit :

```sh
git merge release master --no-commit
git push origin master
```

Si cela échoue, faites un rebase sur master et refaites une compilation :

```sh
git pull
git checkout release
git rebase master
git push origin HEAD
```

## Exécuter une compilation de debug local

Exécutez la version de debug local pour vérifier que vous êtes bien en train de compiler la version que vous souhaitez. Parfois, vous on peux penser que l'on fait une nouvelle version, mais ce n'est pas le cas.

```sh
npm run build
npm start
```

Vérifiez que la fenêtre affiche la version de mise à jour actuelle.

## Définir les variables d'environnement

Vous devrez définir les variables d’environnement suivantes pour publier une nouvelle version. Demandez à un autre membre de l’équipe pour ces informations d’identification.

- `ELECTRON_S3_BUCKET`
- `ELECTRON_S3_ACCESS_KEY`
- `ELECTRON_S3_SECRET_KEY`
- `ELECTRON_GITHUB_TOKEN` - un jeton d'accès personnelle avec le scope "repo".

Vous aurez besoin de le faire seulement une fois.

## Publier la version

Ce script va télécharger les fichiers binaires et générer les en-têtes de node et l’éditeur de liens .lib utilisé sur Windows par node-gyp pour compiler les modules natifs.

```sh
npm run release
```

Remarque: Beaucoup de distributions de Python sont toujours livrés avec de vieux certificats HTTPS. Vous pouvez voir un `InsecureRequestWarning`, mais c'est négligeable.

## Supprimer la branche temporaire

```sh
git checkout master
git branch -D release # delete local branch
git push origin :release # delete remote branch
```

## Promoting a release on npm

New releases are published to npm with the `beta` tag. Every release should eventually get promoted to stable unless there's a good reason not to.

Releases are normally given around two weeks in the wild before being promoted. Before promoting a release, check to see if there are any bug reports against that version, e.g. issues labeled with `version/1.7.x`.

It's also good to ask users in Slack if they're using the beta versions successfully.

To see what's beta and stable at any given time:

    $ npm dist-tag ls electron  
    beta: 1.7.5
    latest: 1.6.11
    

To promote a beta version to stable (aka `latest`):

    npm dist-tag add electron@1.2.3 latest