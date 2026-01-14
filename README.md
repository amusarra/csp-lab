# Content Security Policy Lab

Questo repository è un laboratorio didattico per sperimentare le Content Security Policy (CSP) attraverso una serie di esercizi pratici. Le CSP sono un potente meccanismo di sicurezza web che aiuta a prevenire attacchi come Cross-Site Scripting (XSS) e data injection.

> Nota importante sullo stato iniziale del progetto
>
> - In questa fase iniziale alcuni componenti NON esistono ancora e fanno parte degli esercizi:
>  - `CspFilter` (filtro che genera il nonce e imposta gli header CSP) non è presente: uno degli esercizi è implementarlo.
>  - `CspReportResource` (endpoint che riceve i report CSP) non è presente: va creato come parte dell'esercizio.
>- Per le pagine sotto `src/main/resources/templates/pub/` le policy CSP devono essere applicate tramite `src/main/resources/application.properties` (utile per sperimentare `Content-Security-Policy-Report-Only`).

## Requisiti per l'esecuzione del progetto

Il progetto di questo laboratorio è basato su Quarkus e Java. Per eseguirlo localmente, sono necessari i seguenti prerequisiti:

- Java 17 o superiore installato e configurato (JAVA_HOME). Quarkus 3 richiede Java 17+.
- Maven (è presente il wrapper ./mvnw) o Gradle come tool di build.
- (Opzionale) Quarkus CLI per comandi helper: `quarkus` (non obbligatorio).
- Porta libera (di default Quarkus usa 8080; proprietà: `quarkus.http.port`).
- Browser moderno con strumenti di sviluppo (Console / Network) per osservare le violazioni CSP e i report.
- (Opzionale) Docker, se si preferisce eseguire l'app in un container.

Comandi tipici (usare il wrapper quando disponibile):

- Modalità sviluppo (ricarica automatica):
    - ./mvnw quarkus:dev
- Build:
    - ./mvnw package
- Eseguire il jar prodotto (layout fast-jar di Quarkus):
    - java -jar target/quarkus-app/quarkus-run.jar

## Risorse utili per Quarkus

A seguire alcune risorse utili per lavorare con Quarkus e in particolare con i filtri, risorse REST e Qute (motore di template usato nel progetto):

- Documentazione ufficiale Quarkus: https://quarkus.io/guides/
- Request or response filters: https://quarkus.io/guides/rest#request-or-response-filters
- Qute Templating Engine: https://quarkus.io/guides/qute
- Quarkus All configuration Reference: https://quarkus.io/guides/all-config. Cercare **http.header** per vedere come impostare header HTTP

## Endpoint principali

A seguire sono elencati gli endpoint principali del progetto:

- `http://localhost:8080/no-csp` - endpoint per testare la pagina `src/main/resources/templates/pub/no-csp.html` senza CSP (da analizzare nel primo esercizio)
- `http://localhost:8080/dom-based-xss` - endpoint per testare la pagina `src/main/resources/templates/pub/dom-based-xss.html` (da analizzare nell'esercizio 6)
- `http://localhost:8080/nonce` - endpoint per testare la pagina `src/main/resources/templates/nonce-index.html` con nonce (da completare negli esercizi 3 e 4)
- `http://localhost:8080/csp-report` - endpoint per ricevere i report CSP (da creare nell'esercizio 5)

## Descrizione degli esercizi

1) Analizzare `src/main/resources/templates/pub/no-csp.html`
   - Obiettivo: capire quali risorse vengono caricate e quali script vengono eseguiti senza alcuna restrizione.
   - Cose da verificare
     - il font `awsone.woff2` viene caricato;
     - gli script inline ed esterni vengono eseguiti senza problemi.
   - Strumenti: usare la console del browser (F12) per osservare le richieste di rete e gli script eseguiti.

2) Partendo dalla pagina `src/main/resources/templates/pub/no-csp.html`, identificare le risorse legittime e gli script inline che configurano un possibile attacco XSS.
   - Obiettivo: creare una CSP che permetta il funzionamento delle risorse legittime ma blocchi gli script inline non autorizzati.
   - Cose da verificare
     - il font `awsone.woff2` viene caricato; 
     - gli script inline senza nonce sono bloccati;
     - gli script legittimi (con nonce o da `self`) funzionano.
   - Strumenti: usare la console del browser per osservare le violazioni CSP.
   - Nota: in questa fase non è ancora necessario implementare il filtro o il nonce; si tratta di progettare la policy e usare `application.properties` per applicarla.

3) Implementare `CspFilter` e far funzionare `src/main/resources/templates/nonce-index.html` sotto una CSP sicura
   - Obiettivo: il filtro deve generare un nonce per ogni richiesta, impostare gli header CSP (come per esempio `X-CSP-Nonce`) e rendere disponibile il nonce al template.
   - Cose da verificare
     - il font `awsone.woff2` viene caricato;
     - il css viene caricato correttamente;
     - gli script inline senza nonce sono bloccati;
     - gli script legittimi (con nonce o da `self`) funzionano.
   - Strumenti: usare la console del browser per osservare le violazioni CSP.
   - Suggerimento: il filtro deve generare un nonce sicuro (stringa casuale per esempio una UUID) e inserirlo nella request/context prima del rendering del template.
   - Nota: l'esercizio richiede di completare `NonceResource` per leggere il nonce generato dal filtro. Vedi esercizio successivo.

4) Completare `NonceResource` e `src/main/resources/templates/nonce-index.html` per usare il nonce generato dal filtro
   - Obiettivo: assicurarsi che il nonce generato dal server sia disponibile nel template prima del rendering.
   - Nota: `NonceResource` deve gestire correttamente il caso in cui il nonce non sia ancora presente (fallback, logging) durante lo sviluppo.
   - Cose da verificare
     - uno script con `nonce="{nonce}"` viene eseguito;
     - uno style con `nonce="{nonce}"` viene applicato;
     - uno script inline senza nonce viene bloccato.
   - Nota: questo esercizio è strettamente legato al precedente; il filtro deve essere eseguito prima del controller per garantire che il nonce sia disponibile.
   - Suggerimento: controllare il flusso di esecuzione per assicurarsi che il filtro venga eseguito prima del controller.
   - Strumenti: usare la console del browser per osservare le violazioni CSP.

5) Creare `CspReportResource` e abilitare la raccolta dei report (report-only)
   - Obiettivo: inviare i violation report a `/csp-report` senza bloccare (usare `Content-Security-Policy-Report-Only`) e analizzare i report ricevuti.

6) Analizzare `src/main/resources/templates/pub/dom-based-xss.html` (DOM-based XSS)
   - Obiettivo: riprodurre l'attacco usando `innerHTML`, capire perché `textContent` è sicuro e come una CSP può mitigare il rischio.
   - Cose da verificare
     - caricare la pagina con un hash contenente `<img src=x onerror=alert(1)>` e osservare cosa viene eseguito prima e dopo l'applicazione della CSP.
   - Nota: applicare la CSP tramite `application.properties`
   - Strumenti: usare la console del browser per osservare le violazioni CSP e il comportamento della pagina.

7) Specificare le direttive per risorse particolari
   - Obiettivo: capire come `font-src`, `img-src`, `connect-src`, `style-src` influenzano i fallback da `default-src`.
   - Cose da verificare: aggiungere `font-src 'self' data:` o altra policy e osservare la console del browser per i messaggi di violazione.

File utili/punti di partenza

Classi Java da creare/completare:
- `src/main/java/github/amusarra/csp/filter/CspFilter.java` — DA CREARE come esercizio: filtro che costruisce la policy e genera il nonce.
- `src/main/java/github/amusarra/csp/resources/reports/CspReportResource.java` — DA CREARE come esercizio: endpoint che riceve i report CSP.
- `src/main/java/github/amusarra/csp/resources/NonceResource.java` — presente: va completato per gestire correttamente la lettura del nonce.

Template HTML: 
- `src/main/resources/templates/pub/no-csp.html`, `src/main/resources/templates/pub/dom-based-xss.html`, `src/main/resources/templates/nonce-index.html`.
- `src/main/resources/application.properties` da usare per applicare policy di prova alle pagine sotto `templates/pub/`.

## Risorse Aggiuntive

Riferimenti e strumenti utili per approfondire Content Security Policy (CSP):

- OWASP — Content Security Policy Cheat Sheet  
  https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html
- W3C — Content Security Policy (specifica)  
  https://www.w3.org/TR/CSP3/
- MDN Web Docs — Content Security Policy (guida e riferimenti)  
  https://developer.mozilla.org/docs/Web/HTTP/CSP
- Google CSP Evaluator — strumento per analizzare le policy CSP  
  https://csp-evaluator.withgoogle.com/
- Report-uri / Report collection tools — servizi e strumenti per raccogliere report CSP  
  https://report-uri.com/ (o usare endpoint locali come `/csp-report`)
- CSP Mitigations & Reporting (W3C) — dettagli su reporting e meccanismi correlati  
  https://www.w3.org/TR/CSP3/#reporting

Suggerimento: consultare queste risorse per costruire policy robuste, testarle in modalità `report-only` e comprendere i trade-off tra sicurezza e funzionalità dell'applicazione.

## Licenza

Questo progetto è rilasciato sotto la licenza MIT per scopi educativi. Vedi il file [LICENSE](LICENSE.md) per i dettagli.

## Disclaimer

Questo laboratorio è fornito "così com'è" senza garanzie di alcun tipo. L'autore non è responsabile per eventuali danni derivanti dall'uso del codice o delle informazioni contenute in questo repository.
