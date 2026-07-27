# Projeto 3 — PWA

Esta versão mantém o dashboard original e troca somente a camada de comunicação
pela API JSONP do Google Apps Script.

## 1. Alteração mínima no Apps Script

No projeto atual, substitua **somente** a função `doGet()` por esta versão.
Mantenha `getDashboardData()` e todas as demais funções como estão:

```javascript
function doGet(e) {
  const callback = e && e.parameter ? e.parameter.callback : '';

  try {
    const payload = JSON.stringify({
      ok: true,
      data: getDashboardData()
    });

    if (callback) {
      if (!/^[A-Za-z_$][0-9A-Za-z_$]*$/.test(callback)) {
        throw new Error('Callback inválido.');
      }
      return ContentService
        .createTextOutput(callback + '(' + payload + ');')
        .setMimeType(ContentService.MimeType.JAVASCRIPT);
    }

    return ContentService
      .createTextOutput(payload)
      .setMimeType(ContentService.MimeType.JSON);
  } catch (error) {
    const payload = JSON.stringify({
      ok: false,
      error: error && error.message ? error.message : String(error)
    });

    if (callback && /^[A-Za-z_$][0-9A-Za-z_$]*$/.test(callback)) {
      return ContentService
        .createTextOutput(callback + '(' + payload + ');')
        .setMimeType(ContentService.MimeType.JAVASCRIPT);
    }

    return ContentService
      .createTextOutput(payload)
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

Depois, atualize a implantação existente em **Implantar > Gerenciar
implantações**. Execute como você e permita acesso aos usuários que abrirão o
dashboard. Copie a URL terminada em `/exec`.

## 2. URL da API

O `index.html` já está configurado para consumir:

```text
https://script.google.com/macros/s/AKfycbwbK9YLDBm7zBodVpLd2A2g-g9tZcWilPWutz3w9LiS9VqYhmbyHPzcHSZQNglZLCEOWg/exec?api=1
```

## 3. Publicar no GitHub Pages

Envie `index.html`, `manifest.webmanifest`, `service-worker.js` e a pasta
`icons` para a raiz publicada pelo GitHub Pages. Não envie os arquivos do Apps
Script ao GitHub Pages.

O service worker armazena somente arquivos estáticos (arquivos locais, fontes e
a biblioteca de gráficos). Requisições ao Apps Script e ao domínio de resposta
do Google usam `cache: "no-store"` e também recebem um parâmetro variável,
portanto os dados financeiros não entram no cache da PWA.

## 4. Instalação

- Android/Chrome: use **Instalar app** ou **Adicionar à tela inicial**.
- Computador/Chrome ou Edge: use o ícone de instalação na barra de endereço.
- iPhone/iPad/Safari: use **Compartilhar > Adicionar à Tela de Início**. O
  Safari iOS não exibe o mesmo aviso automático de instalação do Chrome.

Para validar uma nova publicação, abra o DevTools e confira em
**Application > Manifest** e **Application > Service Workers**.
