# ✅ Guida completa all'integrazione di CookieYes (free) con Google Tag Manager e Google Analytics 4

Questa guida dettagliata illustra **ogni passaggio** per integrare **CookieYes CMP gratuito** con **Google Tag Manager** e far sì che **Google Analytics 4 venga attivato solo dopo consenso esplicito ai cookie analitici**.

---

## 🔢 FASE 1 – Registrazione e configurazione iniziale su CookieYes

1. Vai su [https://app.cookieyes.com](https://app.cookieyes.com)
2. Registrati gratuitamente.
3. Clicca su “Add Website” e inserisci l’URL del tuo sito (es. `https://www.iltuosito.it`).

---

## 🛠️ FASE 2 – Attiva il supporto Google Consent Mode in CookieYes

1. Nel menu, vai su **Cookie Banner > Support GCM**.
2. Attiva:
   - ✅ “Support Google Consent Mode”
3. Disattiva:
   - ❌ “Allow Google tags to fire before consent”
4. Salva le modifiche.

---

## 🧾 FASE 3 – Ottieni la Website Key per GTM (da CookieYes)

⚠️ Sei ancora dentro [https://app.cookieyes.com](https://app.cookieyes.com)

1. Nel menu laterale sinistro, vai su **Advanced Settings**.
2. Clicca su **Get Installation Code**.
3. Si aprirà una finestra che mostra uno script simile a questo:

```html
<script id="cookieyes" type="text/javascript" src="https://cdn-cookieyes.com/client_data/09hfbg5cd56cec161c706c11/script.js"></script>
```

4. **La tua Website Key è la parte tra** `client_data/` **e** `/script.js`:
   - In questo esempio sarebbe: `09hfbg5cd56cec161c706c11`
5. Copia questa chiave: ti servirà successivamente per configurare il tag CookieYes CMP in Google Tag Manager.

---

## 🧮 FASE 4 – Crea le variabili in Google Tag Manager

### Variabile 1 – `analyticscookies`

- Vai su **Variabili > Nuova**
- Nome: `analyticscookies`
- Tipo: **Variabile JavaScript personalizzata**
- Codice:

```javascript
function() {
  var consent = {{Cookie CookieYes Consent}};
  if (consent && consent.includes("analytics:yes")) {
    return "granted";
  } else {
    return "denied";
  }
}
```

---

### Variabile 2 – `Cookie CookieYes Consent`

- Vai su **Variabili > Nuova**
- Nome: `Cookie CookieYes Consent`
- Tipo: **Cookie di prima parte**
- Nome cookie: `cookieyes-consent`

---

## 🎯 FASE 5 – Crea i trigger (attivatori)

### Trigger 1 – `GA4 - Dopo consenso`

- Vai su **Trigger > Nuovo**
- Nome: `GA4 - Dopo consenso`
- Tipo di trigger: **Evento personalizzato**
- Nome evento: `cookie_consent_update`
- Attivazione: **Alcuni eventi personalizzati**
- Condizione:
  - Variabile: `analyticscookies`
  - Operatore: uguale a
  - Valore: `granted`

---

### Trigger 2 – `GA4 - Se già consentito`

- Vai su **Trigger > Nuovo**
- Nome: `GA4 - Se già consentito`
- Tipo di trigger: **Inizializzazione del consenso**
- Attivazione: **Alcuni eventi**
- Condizione:
  - Variabile: `analyticscookies`
  - Operatore: uguale a
  - Valore: `granted`


---

## 🧩 FASE 6 – Aggiungi il tag **CookieYes CMP** da galleria GTM

1. Vai su **Tag > Nuovo**
2. Nome: `CookieYes CMP`
3. Clicca su **Sfoglia galleria tag**
4. Cerca `CookieYes CMP` e selezionalo
5. Inserisci la **Website Key** salvata alla FASE 3
6. Attivatore: `All Pages`
7. Salva

---

## 📊 FASE 7 – Crea il tag Google Analytics 4 e assegna i trigger

1. Vai su **Tag > Nuovo**
2. Nome: `GA4 - page_view + consent`
3. Tipo: **Tag HTML personalizzato**
4. Inserisci il seguente codice:

```html
<script>
  window.dataLayer = window.dataLayer || [];

  function loadGA4() {
    var s = document.createElement('script');
    s.src = 'https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXX';
    s.async = true;
    document.head.appendChild(s);

    window.dataLayer.push({ 'gtm.start': new Date().getTime(), event: 'gtm.js' });

    window.gtag = function(){ dataLayer.push(arguments); };

    gtag('consent', 'default', {
      ad_storage: 'denied',
      analytics_storage: 'denied',
      wait_for_update: 500
    });

    gtag('js', new Date());
    gtag('config', 'G-XXXXXXXX', {
      page_path: location.pathname
    });
  }

  if (document.cookie.includes('analytics:yes')) {
    loadGA4();
  }
</script>
```

5. **Sezione Attivazione**:
   - Clicca su **Aggiungi Trigger**
   - Seleziona i trigger:
     - `GA4 - Dopo consenso`
     - `GA4 - Se già consentito`

6. Salva il tag.


Nello script sostituisci `G-XXXXXXXX` con il tuo ID Google Analytics 4.

---

## 🔍 FASE 8 – Verifica funzionale completa tramite Tag Assistant

### 1. Entra in modalità **Anteprima** di Google Tag Manager

- Dal pannello principale di GTM, clicca su **Anteprima**.
- Inserisci l’URL del tuo sito e clicca su **Connetti**.

### 2. Si apre **Tag Assistant**: controlla che il sito risulti connesso.

### 3. Testa la navigazione SENZA consenso ai cookie analitici

- Sul sito, **non accettare subito i cookie** o clicca su **Rifiuta tutti** se presente.
- Naviga normalmente una o più pagine.

### 4. Controlla in **Tag Assistant**

- Nell’evento iniziale (`Page View`):
  - **Tab "Tag"**:
    - Controlla che il tag `GA4 - page_view + consent` **NON sia stato attivato**.
  - **Tab "Variabili"**:
    - Cerca la variabile `analyticscookies`.
    - Deve risultare: `denied`.
  - **Tab "Consenso"**:
    - Valori previsti:
      - `analytics_storage: denied`
      - `ad_storage: denied`

---

### 5. Testa la navigazione DOPO consenso ai cookie analitici

- Clicca sul banner CookieYes su **Personalizza preferenze** (o equivalente).
- **Abilita** la categoria dei **cookie analitici**.
- **Salva** le preferenze.

### 6. Verifica di nuovo in **Tag Assistant**

- Nell’evento `cookie_consent_update` che si genera:
  - **Tab "Tag"**:
    - Il tag `GA4 - page_view + consent` deve **attivarsi**.
  - **Tab "Variabili"**:
    - La variabile `analyticscookies` deve risultare ora: `granted`.
  - **Tab "Consenso"**:
    - Valori previsti:
      - `analytics_storage: granted`
      - `ad_storage: denied`

---

### 7. Opzionale: Disabilita nuovamente i cookie analitici

- Torna al banner CookieYes.
- Disabilita i cookie analitici.
- Salva.

### 8. Verifica cosa succede

- Controlla in **Tag Assistant** sull’evento successivo (`cookie_consent_update`):
  - **Tab "Tag"**:
    - Il tag `GA4 - page_view + consent` **non dovrebbe più attivarsi**.
  - **Tab "Variabili"**:
    - La variabile `analyticscookies` torna a `denied`.
  - **Tab "Consenso"**:
    - `analytics_storage: denied`

---

## ✅ Se tutti i test sono superati:
- GA4 si attiva solo **dopo consenso esplicito**.
- Sei conforme a GDPR + Google Consent Mode v2.

---

## 🧪 FASE 9 – Test banner CookieYes e categorie

1. Vai su **Cookie Manager** in CookieYes
2. Lancia una **scansione manuale** del sito
3. Se non vengono rilevati cookie analitici, aggiungine uno fittizio
4. Pubblica le modifiche
5. Ora gli switch compariranno

---

## 🧠 FASE 10 – Verifica Livello Dati e stato consenso

1. In **Tag Assistant**, clicca sull’evento `cookie_consent_update`
2. Nel tab **Variabili**, controlla: `analyticscookies == granted`
3. Nel tab **Consenso**, verifica: `analytics_storage: granted`

---

## ✅ FASE 11 – Validazione finale

| Caso                          | analyticscookies | GA4 parte? |
|-------------------------------|------------------|------------|
| Nessun consenso dato          | denied           | ❌ No      |
| Solo consenso “Necessari”     | denied           | ❌ No      |
| Accettati “Analitici”         | granted          | ✅ Sì       |

---

## 💾 FASE 12 – Backup e consigli finali

1. Vai su **Amministrazione GTM**
2. Esporta il contenitore `.json` per backup

---

## 🎉 Fine!

Hai configurato:
- CookieYes CMP gratuito
- Google Consent Mode v2
- Google Analytics 4 solo dopo consenso

Ora sei **conforme a GDPR** e **Google Mode V2** ✅
