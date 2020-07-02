# Configuration avancée

Alors que Starship est un shell polyvalent, vous devez parfois faire plus que d'éditer `starship.toml` pour le faire les choses. Cette page détaille quelques techniques de configuration avancées qu'on peut utiliser dans starship.

::: warning

Les configurations de cette section sont sujettes à modification dans les versions à venir de Starship.

:::

## Commandes pré-commande et pré-exécution personnalisées en Bash

Bash n'a pas de cadre officiel préexec/précmd comme la plupart des autres shell du commande. C'est pourquoi il est difficile de fournir des crochets entièrement personnalisables dans `bash`. Cependant, Starship vous donne une capacité limitée à insérer vos propres fonctions dans la procédure de rendu commande :

- Pour exécuter une fonction personnalisée juste avant que commande soit dessinée, définissez une nouvelle fonction et assignez son nom à `starship_precmd_user_func`. Par exemple, pour dessiner une fusée avant la commande, vous feriez

```bash
function blastoff(){
    echo "🚀"
}
starship_precmd_user_func="blastoff"
```

- Pour exécuter une fonction personnalisée juste avant l'exécution d'une commande, vous pouvez utiliser le [` DEBUG` mécanisme d'interruption ](https://jichu4n.com/posts/debug-trap-and-prompt_command-in-bash/). Cependant, vous **devez** pièger le signal DEBUG *avant* initialisation du Starship ! Starship peut préserver la valeur du piège DEBUG, mais si le piège est écrasé après le démarrage de Starship, certaines fonctionnalités vont casser.

```bash
function blastoff(){
    echo "🚀"
}
trap blastoff DEBUG     # Pièger DEBUG *avant* l'initiation de starship
eval $(starship init bash)
```

## Modifier le style des fenêtres commande

Certaines commandes du shell changeront automatiquement le titre de la fenêtre (par exemple, refléter votre répertoire de travail). Fish le fait par défaut. Starship ne le fait pas, mais il est assez simple d'ajouter cette fonctionnalité à `bash` ou `zsh`.

Tout d'abord, définir une fonction de changement de titre de fenêtre (identique en bash et zsh) :

```bash
function set_titre_fenetre(){
    echo -ne "\033]0; TON_TITRE_FENETRE_ICI \007"
}
```

Vous pouvez utiliser des variables pour personnaliser ce titre (`$USER`, `$HOSTNAME`, et `$PWD` sont des choix populaires).

Dans `bash`, définissez cette fonction comme la fonction précmd Starship :

```bash
starship_precmd_user_func="set_titre_gagnante"
```

Dans `zsh`, ajoutez ceci au tableau `precmd_functions` :

```bash
précmd_functions+=(set_titre_gagnant)
```

If you like the result, add these lines to your shell configuration file (`~/.bashrc` or `~/.zshrc`) to make it permanent.

For example, if you want to display your current directory in your terminal tab title, add the following snippet to your `~/.bashrc` or `~/.zshrc`:

```bash
function set_win_title(){
    echo -ne "\033]0; $(basename $PWD) \007"
}
starship_precmd_user_func="set_win_title"
```

## Chaînes de style

Style strings are a list of words, separated by whitespace. The words are not case sensitive (i.e. `bold` and `BoLd` are considered the same string). Each word can be one of the following:

  - `bold`
  - `underline`
  - `dimmed`
  - `bg:<color>`
  - `fg:<color>`
  - `<color>`
  - `none`

where `<color>` is a color specifier (discussed below). `fg:<color>` and `<color>` currently do the same thing , though this may change in the future. The order of words in the string does not matter.

The `none` token overrides all other tokens in a string, so that e.g. `fg:red none fg:blue` will still create a string with no styling. It may become an error to use `none` in conjunction with other tokens in the future.

A color specifier can be one of the following:

 - Une des couleurs standard du terminal : `black`, `red`, `green`, `blue`, `yellow`, `purple`, `cyan`, `white`. Vous pouvez éventuellement les préfixer avec `bright-` pour obtenir la version lumineuse (par exemple `bright-white`).
 - Un `#` suivi d'un nombre hexadécimal de six chiffres. Ceci spécifie un [ Code hexadécimal de couleur RVB ](https://www.w3schools.com/colors/colors_hexadecimal.asp).
 - Un nombre entre 0-255. Ceci spécifie un [code de couleur ANSI 8 bits](https://i.stack.imgur.com/KTSQa.png).

If multiple colors are specified for foreground/background, the last one in the string will take priority.
