# org-actions

## Descrizione generale

La repository in questione è stata pensata per raggruppare tutte le action che verranno utilizzate all'interno dell'organizzazione.

La struttura attuale della cartella è:
```bash
.
├── README.md
└── shared_actions
   ├── issues_slack_notify.yml
   ├── pr_slack_notify.yml
   └── sync_submodule_action.yml
```

È fondamentale che tutte le action presenti in `shared_actions` siano funzionanti e non presentino errori. Nel caso in cui questo vincolo non sia rispettato la action malfunzionante verrà propagata all'interno di tutte le repository che utilizzano prg-actions come submodule

## Utilizzo all'interno delle altre repository dell'organizzazione

Come prima cosa è necessario aggiungere all'interno della repository un submodule:
```bash
git submodule add https://github.com/unipr-org/org-actions actions_submodule
```

> Vedi [wiki git](https://git-scm.com/book/en/v2/Git-Tools-Submodules)

Successivamente bisogna copiare il file `shared_actions/sync_submodule_action.yml` all'interno della folder `.github/workflows/`

> Il posizionamento del file `sync_submodule_action.yml` all'interno della cartella shared_actions è intenzionale. Ogni volta che una repository aggiorna le action "ereditate" dalla repository "org-actions" andrà ad aggiornare anche quella che si occupa della sincronizzazione

_contenuto del file sync_submodule_action.yml_
```bash
name: Sync submodule

on:
  push:
    branches:
      - main
permissions:
  actions: write
  contents: write
  
jobs:
  sync-submodule:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3
        with:
          token: ${{ secrets.WORKFLOW_TOKEN }}

      - name: Aggiunta variabili ambiente
        run: |
          git config user.name github-actions
          git config user.email github-actions@github.com
      
      - name: Rimozione file
        run: |
          ls -la .github/workflows/
          find ./org-actions/shared_actions/ -type f -exec sh -c '
            file="{}"
            destination="./.github/workflows/$(basename "$file")"
            [ -e "$destination" ] && rm "$destination"
          ' \;
          ls -la .github/workflows/
          
      - name: Aggiornamento submodule
        run: |
          git submodule update --init --recursive
          git submodule update --recursive --remote
          
      - name: Copia file da actions_submodule
        run: |
          cp -r ./actions_submodule/shared_actions/* ./.github/workflows/
          git add ./.github/workflows/
          git diff --exit-code --quiet || git commit -m "@sync_submodule_action: Copia automatica di file da ./org-actions/shared_actions/* a ./.github/workflows/"
      
      - name: Push submodule
        run: |
          git push origin main
```

In questo modo ad ogni evento di tipo "push", la repository a cui è stata aggiunta questa action verrà aggiornata con gli ultimi moduli messi a disposizione all'interno della folder `shared_modules`

> secrets.WORKFLOW_TOKEN è un access token privato
### Actions permissions

Potrebbe essere necessario modificare i permessi per delle action all'interno della repository. Andare in `Settings -> Actions -> General -> Actions permissions` e selezionare la voce "Allow all actions and reusable workflows"

## Actions presenti

### Actions slack

#### issues_slack_notify
 
 La action `issues_slack_notify` si occupa di inviare ad un canale Slack la notifica di creazione di una nuova issue.

Contenuto della notifica:

| Contenuto           | Keyword                              |
| ------------------- | ------------------------------------ |
| Link alla issue     | `${{ github.event.issue.html_url }}` |
| Autore della issue  | `${{ github.actor }}`                |
| Repository soggetta | `${{ github.repository }}`           |
| Titolo della issue  | `{{ github.event.issue.title }}`     |

#### pr_slack_notify

 La action `pr_slack_notify` si occupa di inviare ad un canale Slack la notifica di apertura di una nuova pull request.

Contenuto della notifica:

| Contenuto           | Keyword                              |
| ------------------- | ------------------------------------ |
| Link alla pr     | `${{ github.event.pull_request.html_url }}` |
| Autore della pr  | `${{ github.actor }}`                |
| Repository soggetta | `${{ github.repository }}`           |
| Titolo della pr  | `{{ github.event.pull_request.title }}`     |


> Per maggiori informazioni [Slack interaction](https://github.com/marketplace/actions/post-slack-message "https://github.com/marketplace/actions/post-slack-message")
> Per la creazione della struttura della notifica [Slack Block Kit Builder](https://api.slack.com/block-kit-builder/)

## Altri link

Per avere maggiori informazioni su:
- [Secrets](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions)
- [Access token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)


## Contributors

<a href="https://github.com/unipr-org/org-actions/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=unipr-org/org-actions" />
</a>