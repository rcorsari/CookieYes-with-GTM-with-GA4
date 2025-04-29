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

## 🧾 FASE 3 – Ottieni la Website Key per GTM (su CookieYes)

⚠️ Sei ancora dentro il sito [https://app.cookieyes.com](https://app.cookieyes.com)

1. Dal menu laterale sinistro, vai su **Advanced Settings > Google Tag Manager**
2. Copia la tua **Website Key** (stringa tipo `abc123...`)
3. Questa chiave ti servirà tra poco in GTM, **non chiudere la pagina**

---

## 🧮 FASE 4 – Crea le variabili in Google Tag Manager

### Variabile 1 – `analyticscookies`

- Vai su **Variabili > Nuova**
- Nome: `analyticscookies`
- Tipo: **Variabile JavaScript personalizzata**
- Codice:
```javascript
function() {
  var match = document.cookie.match(/cookieyes-consent=([^;]+)/);
  if (match) {
    var consent = decodeURIComponent(match[1]);
    return consent.includes("analytics:yes") ? "granted" : "denied";
  }
  return "denied";
}
```

### Variabile 2 – `Cookie CookieYes Consent`

- Tipo: **Cookie di prima parte**
- Nome: `cookieyes-consent`

---

## 🎯 FASE 5 – Crea i trigger (attivatori)

### Trigger 1 – `GA4 - Dopo consenso`

- Tipo: **Evento personalizzato**
- Nome evento: `cookie_consent_update`
- Condizione:
```
analyticscookies uguale a granted
```

### Trigger 2 – `GA4 - Se già consentito`

- Tipo: **Inizializzazione del consenso**
- Condizione:
```
analyticscookies uguale a granted
```

---

## 🧩 FASE 6 – Aggiungi il tag **CookieYes CMP** da galleria GTM

1. Vai su **Tag > Nuovo**
2. Nome: `CookieYes CMP`
3. Clicca su **Sfoglia galleria tag**
4. Cerca `CookieYes CMP` e selezionalo
5. Inserisci la **Website Key**
6. Attivatore: `All Pages`
7. Salva

---

## 📊 FASE 7 – Crea il tag Google Analytics 4 (custom HTML)

1. Vai su **Tag > Nuovo**
2. Nome: `GA4 - page_view + consent`
3. Tipo: **Tag HTML personalizzato**
4. Incolla il seguente codice:

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

Sostituisci `G-XXXXXXXX` con il tuo ID Google Analytics 4.

---

## 🔗 FASE 8 – Collega i due trigger al tag GA4

1. Apri il tag `GA4 - page_view + consent`
2. Nella sezione **Attivazione**, clicca su `+`
3. Aggiungi:
   - `GA4 - Dopo consenso`
   - `GA4 - Se già consentito`
4. Salva

---

## 🚀 FASE 9 – Pubblica e testa in modalità anteprima

1. In GTM, clicca su **Anteprima**
2. Inserisci il tuo sito
3. Apri il sito in nuova finestra
4. Non accettare i cookie → GA4 **non parte**
5. Accetta i cookie analitici → GA4 **parte**
6. Verifica in Tag Assistant che `analyticscookies` sia passato a `granted`
7. Poi clicca su **Pubblica** in GTM

---

## 🧪 FASE 10 – Test banner CookieYes e categorie

1. Vai su **Cookie Manager** in CookieYes
2. Lancia una **scansione manuale** del sito
3. Se non vengono rilevati cookie analitici, aggiungine uno fittizio
4. Pubblica le modifiche
5. Ora gli switch compariranno

---

## 🧠 FASE 11 – Verifica Livello Dati e stato consenso

1. In **Tag Assistant**, clicca sull’evento `cookie_consent_update`
2. Nel tab **Variabili**, controlla: `analyticscookies == granted`
3. Nel tab **Consenso**, verifica: `analytics_storage: granted`

---

## ✅ FASE 12 – Validazione finale

| Caso                          | analyticscookies | GA4 parte? |
|-------------------------------|------------------|------------|
| Nessun consenso dato          | denied           | ❌ No      |
| Solo consenso “Necessari”     | denied           | ❌ No      |
| Accettati “Analitici”         | granted          | ✅ Sì       |

---

## 💾 FASE 13 – Backup e consigli finali

1. Vai su **Amministrazione GTM**
2. Esporta il contenitore `.json` per backup

---

## 🎉 Fine!

Hai configurato:
- CookieYes CMP gratuito
- Google Consent Mode v2
- Google Analytics 4 solo dopo consenso

Ora sei **conforme a GDPR** e **Google Mode V2** ✅
