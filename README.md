✅ 1. Pré-requisitos instalados no PC
Você deve ter:

Requisito	Como verificar
Python 3.9+	python --version no terminal
Google Chrome instalado	Já vem em muitos PCs
WebDriver do Chrome (chromedriver.exe)	Baixe da versão do seu Chrome: https://chromedriver.chromium.org/downloads
Chrome com perfil ativo no WhatsApp Web	Use o perfil onde você já está logado no WhatsApp
Conta Google e uma planilha chamada "Mercado livre" com permissões de edição pela API	

🔧 2. Instale as bibliotecas Python
Abra o terminal (cmd, PowerShell ou VSCode Terminal) na pasta do projeto e rode:

bash
Copiar
Editar
pip install selenium gspread oauth2client requests
📁 3. Coloque os arquivos certos nas pastas certas
Crie um arquivo credenciais.json (credenciais da conta Google Service, veja abaixo como gerar)

Deixe o chromedriver.exe em C:\Users\user\Desktop\chromedirver\chromedriver.exe (ou ajuste no código)

Mantenha o script Python (por exemplo: bot_produtos.py) salvo com o conteúdo que você me mandou

🔑 4. Gerar credenciais.json para Google Sheets
Vá para: https://console.cloud.google.com/

Crie um projeto novo

Ative a API do Google Sheets e API do Google Drive

Vá em Credenciais > Criar credenciais > Conta de serviço

Baixe o arquivo .json e renomeie para credenciais.json

Compartilhe a planilha "Mercado livre" com o e-mail da conta de serviço (xxxxx@xxxxx.iam.gserviceaccount.com)

🌐 5. Configurar o Chrome para usar o WhatsApp Web com sessão ativa
Você vai usar um perfil já logado no WhatsApp Web. Faça isso:

Feche todos os Chrome abertos

Rode esse comando no terminal (ajuste o caminho do Chrome se necessário):

bash
Copiar
Editar
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222 --user-data-dir="C:\Users\user\AppData\Local\Google\Chrome\User Data" --profile-directory="Profile 1"
Isso abrirá o Chrome no modo depuração com o perfil “Profile 1” (altere para outro se necessário)

Verifique se o WhatsApp Web já está logado nesse perfil

🧠 6. Adicione sua chave Groq (IA)
No topo do script, já está definido:

python
Copiar
Editar
GROQ_API_KEY = "gsk_..."
⚠️ Verifique se essa é sua chave válida. Se não tiver uma, crie uma conta em https://console.groq.com e gere uma chave da API.

▶️ 7. Execute o script
Rode o script no terminal com:

bash
Copiar
Editar
python bot_produtos.py
Ele fará:

Coleta dos links de produtos do Mercado Livre

Extração de informações e link de afiliado

Reescrita de textos com IA

Envio da imagem gerada pela Meta AI e texto para o grupo do WhatsApp

🧪 8. Testes e ajustes
Caso veja erros:

Verifique os XPaths (o layout do WhatsApp Web pode mudar)

Ajuste o tempo de espera (time.sleep) para sua internet
