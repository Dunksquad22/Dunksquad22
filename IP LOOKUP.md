IP LOOK UP I MADE ENJOY 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Advanced IP Lookup Tool</title>
<meta name="viewport" content="width=device-width, initial-scale=1" />
<style>
  body {
    background: #000;
    color: #0f0;
    font-family: monospace;
    padding: 20px;
    margin: 0;
  }
  h1,h2 {
    color: #0f0;
    font-weight: normal;
  }
  label {
    display: block;
    margin-top: 15px;
  }
  textarea, input[type=text] {
    width: 100%;
    font-family: monospace;
    font-size: 1rem;
    padding: 10px;
    margin-top: 5px;
    background: #111;
    border: 1px solid #0f0;
    color: #0f0;
    box-sizing: border-box;
    resize: vertical;
  }
  button {
    margin-top: 10px;
    background: #0f0;
    color: #000;
    padding: 10px 15px;
    font-weight: bold;
    border: none;
    cursor: pointer;
    margin-right: 10px;
  }
  #output {
    white-space: pre-wrap;
    margin-top: 20px;
    border: 1px solid #0f0;
    padding: 15px;
    min-height: 300px;
    background: #001100;
  }
  .buttons {
    margin-top: 10px;
  }
</style>
</head>
<body>

<h1>Advanced IP Lookup Tool</h1>

<button id="myipBtn">Lookup My Own IP Info</button>

<label for="singleIp">Lookup Single IP or Hostname:</label>
<input type="text" id="singleIp" placeholder="e.g. 8.8.8.8 or example.com" />

<button id="singleLookupBtn">Lookup Single</button>

<label for="batchIps">Batch Lookup (one IP/hostname per line):</label>
<textarea id="batchIps" rows="5" placeholder="e.g.&#10;8.8.8.8&#10;example.com&#10;1.1.1.1"></textarea>

<button id="batchLookupBtn">Lookup Batch</button>

<div class="buttons">
  <button id="copyBtn">Copy Output</button>
  <button id="downloadTxtBtn">Download TXT</button>
  <button id="downloadCsvBtn">Download CSV</button>
  <button id="downloadPdfBtn">Download PDF</button>
</div>

<div id="output">Results will appear here...</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
<script>
  const output = document.getElementById('output');
  const singleIpInput = document.getElementById('singleIp');
  const batchIpsTextarea = document.getElementById('batchIps');

  async function fetchIPinfo(ip) {
    const url = `https://ipapi.co/${encodeURIComponent(ip)}/json/`;
    try {
      const res = await fetch(url);
      if(!res.ok) throw new Error('Network response not OK');
      const data = await res.json();
      if(data.error) throw new Error(data.reason || 'Invalid IP');
      return {
        ip: data.ip || '',
        hostname: data.hostname || '',
        city: data.city || '',
        region: data.region || '',
        country: data.country_name || '',
        postal: data.postal || '',
        latitude: data.latitude || '',
        longitude: data.longitude || '',
        timezone: data.timezone || '',
        asn: data.asn || '',
        org: data.org || '',
        isp: data.org || ''
      };
    } catch(e) {
      return {error: e.message, ip};
    }
  }

  function formatInfo(info) {
    if(info.error) {
      return `IP: ${info.ip} - ERROR: ${info.error}`;
    }
    return `IP Address    : ${info.ip}
Hostname      : ${info.hostname}
City          : ${info.city}
Region        : ${info.region}
Country       : ${info.country}
Postal Code   : ${info.postal}
Latitude      : ${info.latitude}
Longitude     : ${info.longitude}
Timezone      : ${info.timezone}
ASN           : ${info.asn}
Organization  : ${info.org}
ISP           : ${info.isp}
-----------------------------`;
  }

  async function lookupMyIP() {
    output.textContent = 'Looking up your IP...';
    try {
      const res = await fetch('https://ipapi.co/json/');
      if(!res.ok) throw new Error('Failed to get your IP info');
      const data = await res.json();
      output.textContent = formatInfo(data);
    } catch (e) {
      output.textContent = 'Error: ' + e.message;
    }
  }

  async function lookupSingle() {
    const ip = singleIpInput.value.trim();
    if(!ip) {
      alert('Enter an IP or hostname to lookup!');
      return;
    }
    output.textContent = 'Looking up...';
    const info = await fetchIPinfo(ip);
    output.textContent = formatInfo(info);
  }

  async function lookupBatch() {
    const ipsRaw = batchIpsTextarea.value.trim();
    if(!ipsRaw) {
      alert('Enter one or more IPs/hostnames (one per line)!');
      return;
    }
    const ips = ipsRaw.split('\n').map(l => l.trim()).filter(Boolean);
    output.textContent = 'Looking up batch... This may take a few seconds.\n\n';
    for(let i = 0; i < ips.length; i++) {
      output.textContent += `Lookup ${i+1}/${ips.length}: ${ips[i]}\n`;
      const info = await fetchIPinfo(ips[i]);
      output.textContent += formatInfo(info) + '\n';
    }
    output.textContent += 'Batch lookup complete.';
  }

  function copyOutput() {
    navigator.clipboard.writeText(output.textContent).then(() => alert('Copied to clipboard!'));
  }

  function downloadTxt() {
    const blob = new Blob([output.textContent], {type: 'text/plain'});
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = `ip_lookup_${Date.now()}.txt`;
    a.click();
  }

  function downloadCsv() {
    // Parse output text to CSV format
    const lines = output.textContent.split('\n');
    let csv = '';
    const keys = ['IP Address', 'Hostname', 'City', 'Region', 'Country', 'Postal Code', 'Latitude', 'Longitude', 'Timezone', 'ASN', 'Organization', 'ISP'];
    csv += keys.join(',') + '\n';

    let row = {};
    for(let line of lines) {
      if(line.startsWith('IP:') && line.includes('ERROR')) {
        // Error line: IP: xxx - ERROR: ...
        const ip = line.match(/IP:\s*(.*?)\s*-/)[1];
        csv += `"${ip}","ERROR","","","","","","","","","",""\n`;
        row = {};
        continue;
      }
      if(line.includes(':')) {
        const [k,v] = line.split(':').map(s => s.trim());
        row[k] = v;
      }
      if(line.startsWith('-----------------------------')) {
        // Write row to csv
        let rowData = keys.map(k => `"${row[k] || ''}"`);
        csv += rowData.join(',') + '\n';
        row = {};
      }
    }

    const blob = new Blob([csv], {type: 'text/csv'});
    const a = document.createElement('a');
    a.href = URL.createObjectURL(blob);
    a.download = `ip_lookup_${Date.now()}.csv`;
    a.click();
  }

  async function downloadPdf() {
    const { jsPDF } = window.jspdf;
    const doc = new jsPDF();
    const text = output.textContent;
    const lines = doc.splitTextToSize(text, 180);
    doc.setFont("Courier");
    doc.setFontSize(10);
    doc.setTextColor(0,255,0);
    doc.text(lines, 10, 10);
    doc.save(`ip_lookup_${Date.now()}.pdf`);
  }

  document.getElementById('myipBtn').addEventListener('click', lookupMyIP);
  document.getElementById('singleLookupBtn').addEventListener('click', lookupSingle);
  document.getElementById('batchLookupBtn').addEventListener('click', lookupBatch);
  document.getElementById('copyBtn').addEventListener('click', copyOutput);
  document.getElementById('downloadTxtBtn').addEventListener('click', downloadTxt);
  document.getElementById('downloadCsvBtn').addEventListener('click', downloadCsv);
  document.getElementById('downloadPdfBtn').addEventListener('click', downloadPdf);
</script>

</body>
</html>

<!---
Dunksquad22/Dunksquad22 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
