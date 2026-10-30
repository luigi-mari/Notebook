# Guida deploy book Quantlab (per me, per le prossime volte)

## Mappa dei fatti

| Cosa | Dove |
|---|---|
| Workspace notebook | `C:/PyLuigi/Py Projects/<nome>.ipynb` |
| Clone della repo del book | `C:/PyLuigi/Py Projects/Quantlab` (remote: `https://github.com/luigi-mari/Notebook.git`, branch **main**) |
| Capitoli + TOC | `Quantlab/chapters/*.ipynb` e `Quantlab/chapters/_toc.yml` |
| Dipendenze CI | `Quantlab/requirements.txt` (root) |
| Deploy | `.github/workflows/deploy.yml`: al push su **main** la CI fa lei `jupyter-book build chapters` e pubblica `chapters/_build/html` su GitHub Pages |
| Sito live | `https://luigi-mari.github.io/Notebook/<capitolo>.html` (URL piatte: nome repo + capitolo, mai segmenti extra) |

**La CI fa la build da sola.** Il `_build` locale serve solo per verificare prima di pushare. Non toccare `deploy.yml`: funziona così com'è (`BASE_URL = /<nome_repo>` è ciò che dà le URL piatte).

## Workflow standard: nuova pagina o capitolo aggiornato

1. **Modifica ed esegui il notebook nel workspace**:
   ```
   cd "C:/PyLuigi/Py Projects" && jupyter nbconvert --to notebook --execute --inplace <nome>.ipynb
   python -c "import nbformat; nb=nbformat.read('<nome>.ipynb', as_version=4); nbformat.validate(nb)"
   ```
   Controllare: 0 errori, immagini renderizzate, numeri nelle letture aggiornati.

2. **Sincronizza la copia del book**: `cp <nome>.ipynb Quantlab/chapters/` (stesso per ogni file dati che il notebook legge dalla sua cartella, es. `inout.xlsx`: se manca in `chapters/`, la build CI fallisce).

3. **Nuovo capitolo**: aggiungi l'entry in `Quantlab/chapters/_toc.yml` (l'ordine = ordine della sidebar). Capitolo già esistente: niente da fare qui.

4. **Build locale di verifica** (ambiente `jbook`, non il Python di default):
   ```
   cd "C:/PyLuigi/Py Projects/Quantlab" && rm -rf chapters/_build && \
     timeout 900 "C:/Users/Louis/anaconda3/envs/jbook/Scripts/jupyter-book.exe" build chapters
   ls chapters/_build/html/<nome>.html   # deve esistere
   ```

5. **Commit e push** — aggiungere solo le path interessate, mai `git add -A` sulla root (ci sono file dell'utente non committati da lasciare soli):
   ```
   cd Quantlab && git add chapters/ && git commit -m "..." && git push
   ```

6. **Verifica live** dopo ~90-120 s (la CI impiega 2-4 min):
   ```
   curl -s -o /dev/null -w '%{http_code}' https://luigi-mari.github.io/Notebook/<nome>.html   # deve dare 200
   ```

## Rimuovere un capitolo

1. Togli l'entry da `_toc.yml`, elimina `chapters/<file>`
2. **Build completa pulita** (`rm -rf chapters/_build` prima): la sidebar è rigenerata e non resta alcun link morto (lezione: una build parziale ha lasciato un 404 nella sidebar)
3. Push, verifica che l'URL vecchia dia 404 e le altre 200

## Regole del notebook (preferenze utente)

- Nessuna em dash (`—`) nel markdown delle celle; prosa naturale, non robotica
- Titoli di sezione brevi: niente frasi, niente numeri ("Track record", "Scorecard"); se serve l'ordine, al massimo `PART 1` / `PART 2`
- Assi y in scala lineare quando sono valori in $ (non log)
- Tabelle di confronto con intestazioni esplicite: ogni colonna dice quale caso rappresenta (QQQ / real case / lookahead), mai righe isolate senza contesto
- Ogni scorecard/print etichetta il caso che sta valutando
- Numeri nelle sezioni "Reading" presi dall'esecuzione reale, non hardcoded
- Niente print di servizio sparsi; una cella = un'idea

## Insidie già incontrate (non ricadervi)

- **URL**: i capitoli stanno in `chapters/` radice. Una sottocartella dentro `chapters/` diventa un segmento dell'URL: vietato.
- **Branch**: pushare solo su `main`; esiste anche un vecchio `master` locale, ignorarlo.
- **Dipendenze nuove** nel notebook → aggiungere in `Quantlab/requirements.txt`, altrimenti la CI fallisce in silenzio.
- **Avvisi git CRLF** (LF→CRLF) su Windows: rumore, si ignora.
- **Formule matematiche**: ogni display `$$...$$` va su un paragrafo a sé (riga vuota prima e dopo), mai incollato alla prosa con un solo a capo: MyST abbina male i `$` dell'inline adiacente e lascia `$$`/`$` grezzi nell'HTML. Verifica locale: nell'HTML buildato nessun `$` fuori da `<span class="math">` / `<div class="math">`.
- **Cache browser**: sidebar "vecchia" sul sito = Ctrl+F5; verificare sempre con `curl` prima di sospettare un problema reale.
- Il file `.ipynb_checkpoints/` non tracciato nella root di Quantlab è dell'utente: non commitare, non eliminare.
