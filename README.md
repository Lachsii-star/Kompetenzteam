<!doctype html>
<html lang="de">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>NCSD – Bußgeldbescheid Generator</title>
  <style>
    :root { font-family: system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif; }
    body { margin: 0; padding: 24px; background: #f6f7f9; }
    .wrap { max-width: 980px; margin: 0 auto; display: grid; gap: 16px; }
    .card { background: white; border-radius: 14px; padding: 16px; box-shadow: 0 8px 24px rgba(0,0,0,.08); }
    h1 { margin: 0 0 8px; font-size: 20px; display:flex; align-items:center; justify-content: space-between; gap:12px; flex-wrap:wrap; }
    .titleLeft{ display:flex; align-items:center; gap:10px; flex-wrap:wrap; }
    h2 { margin: 0 0 8px; font-size: 16px; }
    .grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; }

    label { display: grid; gap: 6px; font-size: 13px; color: #333; }
    input, textarea {
      border: 1px solid #d7dbe2; border-radius: 10px; padding: 10px 12px;
      font-size: 14px; outline: none; background: #fff;
      transition: border-color .15s, box-shadow .15s;
    }
    textarea { min-height: 80px; resize: vertical; }

    /* Pflichtfeld-Markierung */
    .requiredHint { font-size: 12px; color: #6b7280; }
    .fieldError input, .fieldError textarea{
      border-color: #ef4444 !important;
      box-shadow: 0 0 0 3px rgba(239, 68, 68, .15);
    }
    .errorText{
      font-size: 12px;
      color: #ef4444;
      display:none;
    }
    .fieldError .errorText{ display:block; }

    /* Optional-Feld-Markierung (gelb/orange) */
    .optionalHint{
      font-size: 12px;
      color: #92400e;
      background: #fffbeb;
      border: 1px solid #f59e0b;
      padding: 2px 8px;
      border-radius: 999px;
      width: fit-content;
    }
    .optionalField input, .optionalField textarea{
      border-color: rgba(245, 158, 11, .65);
      box-shadow: 0 0 0 3px rgba(245, 158, 11, .12);
    }

    .row { display:flex; gap: 10px; flex-wrap: wrap; align-items: center; }
    button {
      border: 0; border-radius: 10px; padding: 10px 14px; font-weight: 600; cursor: pointer;
      background: #111827; color: white;
    }
    button.secondary { background: #e5e7eb; color: #111827; }
    button.success { background: #16a34a; }
    button.success:disabled { opacity: .6; cursor: not-allowed; }

    .muted { color: #6b7280; font-size: 13px; }
    .out {
      white-space: pre-wrap; font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, "Liberation Mono", monospace;
      font-size: 13px; line-height: 1.35; background: #0b1020; color: #e5e7eb;
      border-radius: 14px; padding: 16px;
    }
    .badge { font-size: 12px; padding: 3px 8px; border-radius: 999px; background: #eef2ff; color: #3730a3; }

    /* Kompetenzteam auffälliger (oben rechts) */
    .ktBadge{
      font-size: 12px;
      padding: 8px 12px;
      border-radius: 999px;
      font-weight: 800;
      letter-spacing: .3px;
      background: linear-gradient(135deg, #111827, #3730a3);
      color: #fff;
      box-shadow: 0 10px 22px rgba(17,24,39,.20);
      border: 1px solid rgba(255,255,255,.18);
      text-transform: uppercase;
      white-space: nowrap;
    }

    #copyStatus{
      font-size: 13px;
      color: #16a34a;
      font-weight: 700;
      display:none;
    }

    @media (max-width: 760px) { .grid { grid-template-columns: 1fr; } }
  </style>
</head>
<body>
  <div class="wrap">
    <div class="card">
      <h1>
        <span class="titleLeft">
          ⭐ NARCO COUNTY SHERIFF DEPARTMENT ⭐
          <span class="badge">Bußgeldbescheid-Generator</span>
        </span>
        <span class="ktBadge">Kompetenzteam</span>
      </h1>

      <p class="muted">
        Datum/Uhrzeit werden automatisch laufend aktualisiert. Pflichtfelder werden rot markiert, wenn sie fehlen.
        Optional-Felder sind gelb/orange markiert.
      </p>

      <div class="grid" style="margin-top:12px;">

        <label>
          Datum (auto – live)
          <input id="date" type="text" readonly />
          <span class="requiredHint">&nbsp;</span>
        </label>

        <label>
          Uhrzeit (auto – live)
          <input id="time" type="text" readonly />
          <span class="requiredHint">&nbsp;</span>
        </label>

        <label>
          Ort / Einsatzgebiet
          <input id="location" type="text" value="Jägerjob Strasse, 1077 PLZ" />
          <span class="requiredHint">&nbsp;</span>
        </label>

        <label id="wrapLimit">
          Zulässige Höchstgeschwindigkeit (km/h) <span class="requiredHint">(Pflicht)</span>
          <input id="limit" type="number" min="0" step="1" value="130" />
          <span class="errorText">Pflichtfeld fehlt oder ist ungültig.</span>
        </label>

        <label id="wrapMeasured">
          Gemessene Geschwindigkeit (km/h) <span class="requiredHint">(Pflicht)</span>
          <input id="measured" type="number" min="0" step="1" placeholder="z.B. 170" />
          <span class="errorText">Bitte eine gemessene Geschwindigkeit eintragen.</span>
        </label>

        <label id="wrapTolerance">
          Toleranzabzug (km/h) <span class="requiredHint">(Pflicht)</span>
          <input id="tolerance" type="number" min="0" step="1" value="10" />
          <span class="errorText">Pflichtfeld fehlt oder ist ungültig.</span>
        </label>

        <!-- Kennzeichen optional + gelb/orange -->
        <label class="optionalField">
          Kennzeichen <span class="optionalHint">Optional</span>
          <input id="plate" type="text" placeholder="z.B. NC-12345" />
          <span class="requiredHint">Wenn leer, wird „FAHRZEUGDATEN“ ausgelassen.</span>
        </label>

        <label id="wrapFine">
          Bußgeldbetrag ($) <span class="requiredHint">(Pflicht)</span>
          <input id="fine" type="number" min="0" step="1" value="15000" />
          <span class="errorText">Bitte einen Bußgeldbetrag eintragen.</span>
        </label>

        <label class="optionalField">
          Name / Halter <span class="optionalHint">Optional</span>
          <input id="holder" type="text" placeholder="z.B. Max Mustermann" />
          <span class="requiredHint">&nbsp;</span>
        </label>

        <label class="optionalField">
          Gezeichnet von <span class="optionalHint">Optional</span>
          <input id="officer" type="text" value="Andrew Petzenstein | SD-126 | Coporal" />
          <span class="requiredHint">&nbsp;</span>
        </label>

        <label style="grid-column: 1 / -1;">
          Messverfahren (Text wie im Bescheid)
          <textarea id="method">Die Geschwindigkeitsmessung erfolgte mobil aus einem gekennzeichneten Streifenfahrzeug des Narco County Sheriff Departments mittels fest verbautem Radarmesssystem (Wraith ARS 2 / wk_wars2x Radar-System).
Die Messung erfolgte stationär aus dem Streifenfahrzeug heraus ohne Verfolgungsfahrt.
Es bestand freie und uneingeschränkte Sicht auf das gemessene Fahrzeug.
Zwischenverkehr oder andere Störquellen waren nicht vorhanden.
Das Fahrzeug wurde visuell eindeutig identifiziert.</textarea>
          <span class="requiredHint">&nbsp;</span>
        </label>
      </div>

      <div class="row" style="margin-top:12px;">
        <button id="btnGenerate">Bescheid erstellen / aktualisieren</button>
        <button class="success" id="btnCopy" disabled>Bescheid kopieren</button>
        <span id="copyStatus">✅ Kopiert!</span>
        <span class="muted" id="calcHint"></span>
      </div>
    </div>

    <div class="card" id="previewCard">
      <h2>Vorschau</h2>
      <div class="out" id="output">Noch kein Bescheid erstellt.</div>
    </div>
  </div>

  <script>
    function pad2(n){ return String(n).padStart(2,"0"); }

    function updateNowLive() {
      const d = new Date();
      const day = pad2(d.getDate());
      const month = pad2(d.getMonth() + 1);
      const year = d.getFullYear();
      const hours = pad2(d.getHours());
      const mins = pad2(d.getMinutes());
      const secs = pad2(d.getSeconds());

      document.getElementById("date").value = `${day}.${month}.${year}`;
      // Sekundengenau, damit es "live" sichtbar ist
      document.getElementById("time").value = `${hours}:${mins}:${secs} Uhr`;
    }

    function numberOrNull(id) {
      const v = document.getElementById(id).value;
      if (v === "" || v === null || v === undefined) return null;
      const n = Number(v);
      return Number.isFinite(n) ? n : null;
    }

    function fmtMoneyUSD(n){
      // 15000 -> 15.000$
      const s = Math.round(n).toString();
      return s.replace(/\B(?=(\d{3})+(?!\d))/g, ".") + " $";
    }

    function setFieldError(wrapperId, hasError){
      const el = document.getElementById(wrapperId);
      if (!el) return;
      el.classList.toggle("fieldError", !!hasError);
    }

    async function copyToClipboard(text){
      if (navigator.clipboard && window.isSecureContext) {
        await navigator.clipboard.writeText(text);
        return;
      }
      const ta = document.createElement("textarea");
      ta.value = text;
      ta.style.position = "fixed";
      ta.style.left = "-9999px";
      document.body.appendChild(ta);
      ta.select();
      document.execCommand("copy");
      document.body.removeChild(ta);
    }

    function generate() {
      const date = document.getElementById("date").value.trim();
      const time = document.getElementById("time").value.trim();

      const location = document.getElementById("location").value.trim();
      const limit = numberOrNull("limit");
      const measured = numberOrNull("measured");
      const tolerance = numberOrNull("tolerance");

      const plate = document.getElementById("plate").value.trim();
      const holder = document.getElementById("holder").value.trim();

      const fine = numberOrNull("fine");
      const officer = document.getElementById("officer").value.trim();
      const method = document.getElementById("method").value.trim();

      // Pflichtfelder (Kennzeichen ist NICHT Pflicht)
      const errMeasured = (measured === null);
      const errFine = (fine === null);
      const errLimit = (limit === null);
      const errTol = (tolerance === null);

      setFieldError("wrapMeasured", errMeasured);
      setFieldError("wrapFine", errFine);
      setFieldError("wrapLimit", errLimit);
      setFieldError("wrapTolerance", errTol);

      const errors = [];
      if (errMeasured) errors.push("Gemessene Geschwindigkeit fehlt.");
      if (errFine) errors.push("Bußgeldbetrag fehlt.");
      if (errLimit) errors.push("Höchstgeschwindigkeit fehlt.");
      if (errTol) errors.push("Toleranzabzug fehlt.");

      const btnCopy = document.getElementById("btnCopy");

      if (errors.length) {
        document.getElementById("output").textContent =
          "❗ Bitte ergänzen:\n- " + errors.join("\n- ");
        document.getElementById("calcHint").textContent = "";
        btnCopy.disabled = true;
        btnCopy.dataset.copyText = "";
        return;
      }

      const usable = Math.max(0, measured - tolerance);
      const over = Math.max(0, usable - limit);

      document.getElementById("calcHint").textContent =
        `Berechnung: verwertbar ${usable} km/h | Überschreitung ${over} km/h`;

      // Fahrzeugdaten-Block nur wenn Plate ODER Holder gesetzt ist
      const includeVehicle = (plate.length > 0) || (holder.length > 0);

      let vehicleBlock = "";
      if (includeVehicle) {
        const plateLine = plate ? `Kennzeichen: ${plate}\n` : "";
        const holderLine = holder ? `Halter: ${holder}\n` : "";
        vehicleBlock =
`〚 FAHRZEUGDATEN 〛

${plateLine}${holderLine}${plateLine || holderLine ? "" : ""}
Das Fahrzeug wurde visuell eindeutig identifiziert.
Es bestand freie Sicht auf das gemessene Fahrzeug ohne Zwischenverkehr oder Störquellen.

`;
      }

      const text =
`⭐ NARCO COUNTY SHERIFF DEPARTMENT ⭐

〚 DATEN ZUM VERSTOSS 〛

Datum: ${date}

Uhrzeit: ${time}
Ort: ${location}

*Im betroffenen Streckenabschnitt gilt eine zulässige Höchstgeschwindigkeit von ${limit} km/h.*

〚 MESSVERFAHREN 〛
${method}

〚 MESSDATEN 〛
Gemessene Geschwindigkeit: **${measured} km/h**
Gesetzlicher Toleranzabzug: **${tolerance} km/h**
Verwertbare Geschwindigkeit: **${usable} km/h**
Tatsächliche Überschreitung: **${over} km/h**

${vehicleBlock}〚 RECHTSFOLGE 〛
Gemäß dem geltenden Bußgeldkatalog vom Staat wird folgende Maßnahme verhängt: 
⠂ Bußgeldbetrag: ${fmtMoneyUSD(fine)}
⠂ Zusätzliche Maßnahmen: Keine

〚 GEZEICHNET VON 〛
${officer}`;

      document.getElementById("output").textContent = text;

      // Copy-Button aktivieren + Text merken
      btnCopy.disabled = false;
      btnCopy.dataset.copyText = text;
    }

    // Live Datum/Uhrzeit: jede Sekunde aktualisieren + Bescheid live nachziehen
    updateNowLive();
    setInterval(() => {
      updateNowLive();
      generate();
    }, 1000);

    // Buttons
    document.getElementById("btnGenerate").addEventListener("click", generate);

    document.getElementById("btnCopy").addEventListener("click", async () => {
      const text = document.getElementById("btnCopy").dataset.copyText || "";
      if (!text) return;
      try{
        await copyToClipboard(text);
        const st = document.getElementById("copyStatus");
        st.style.display = "inline";
        setTimeout(() => st.style.display = "none", 1200);
      } catch(e){
        alert("Kopieren nicht möglich (Browser-Restriktion). Bitte manuell markieren.");
      }
    });

    // Live-Update (Inputs)
    ["location","limit","measured","tolerance","plate","holder","fine","officer","method"]
      .forEach(id => document.getElementById(id).addEventListener("input", generate));
  </script>
</body>
</html>
