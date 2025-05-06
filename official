# (código ajustado: coleta todos os produtos, depois envia imagens e mensagens no WhatsApp com conteúdo simplificado e imagens mais relevantes)
import subprocess
import time
import os
import gspread
import requests
import re
from oauth2client.service_account import ServiceAccountCredentials
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

GROQ_API_KEY = ""
GROQ_API_URL = ""

def remover_emojis_incompativeis(texto):
    return ''.join(c for c in texto if ord(c) <= 0xFFFF)

def limpar_links(texto):
    return re.sub(r'\[([^\]]+)\]\(([^\)]+)\)', r'\2', texto)

def reescrever_com_ia(texto_original):
    prompt = (
        "Você é um redator de WhatsApp. Reescreva a mensagem de forma simples e direta, com o seguinte formato:\n"
        "1. Primeira linha: frase curta e chamativa sobre o produto (sem emojis).\n"
        "2. Segunda linha: **De** R$ valor original\n"
        "3. Terceira linha: **Agora por** R$ valor com desconto\n"
        "4. Quarta linha: **Desconto:** percentual\n"
        "5. Quinta linha: link de compra.\n"
        "Use linguagem informal e destaque em negrito nas palavras-chave.\n"
        "Não escreva introduções ou explicações.\n"
        f"Mensagem original:\n{texto_original}"
    )
    headers = {"Authorization": f"Bearer {GROQ_API_KEY}", "Content-Type": "application/json"}
    payload = {
        "model": "llama3-8b-8192",
        "messages": [{"role": "user", "content": prompt}],
        "temperature": 0.7
    }
    try:
        response = requests.post(GROQ_API_URL, headers=headers, json=payload)
        response.raise_for_status()
        content = response.json()["choices"][0]["message"]["content"].strip()
        mensagem_limpa = re.sub(r'^Here is the rewritten message:\n?', '', content, flags=re.IGNORECASE)
        mensagem_sem_emoji = remover_emojis_incompativeis(mensagem_limpa)
        mensagem_sem_link_markdown = limpar_links(mensagem_sem_emoji)
        return mensagem_sem_link_markdown
    except Exception as e:
        print(f"⚠️ Erro com a IA Groq: {e}")
        return texto_original

def conectar_planilha(nome_planilha):
    scope = ["https://spreadsheets.google.com/feeds", "https://www.googleapis.com/auth/drive"]
    creds = ServiceAccountCredentials.from_json_keyfile_name("credenciais.json", scope)
    return gspread.authorize(creds).open(nome_planilha).sheet1

def enviar_para_sheets(sheet, info):
    try:
        sheet.append_row([
            info["nome_produto"], info["link"], info["preco_original"], info["preco_desconto"],
            info["desconto_percentual"], info.get("link_afiliado", "")
        ])
        print("✅ Enviado para Google Sheets!")
    except Exception as e:
        print("❌ Falha ao enviar para Sheets!", e)

def iniciar_chrome():
    subprocess.Popen([
        "C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe",
        "--remote-debugging-port=9222",
        "--user-data-dir=C:\\Users\\user\\AppData\\Local\\Google\\Chrome\\User Data",
        "--profile-directory=Profile 1"
    ])
    time.sleep(5)

def iniciar_driver():
    options = Options()
    options.add_experimental_option("debuggerAddress", "127.0.0.1:9222")
    service = Service("C:\\Users\\user\\Desktop\\chromedirver\\chromedriver.exe")
    return webdriver.Chrome(service=service, options=options)

def coletar_links_da_pagina(driver, url):
    driver.get(url)
    WebDriverWait(driver, 10).until(EC.presence_of_all_elements_located((By.CLASS_NAME, "poly-card__content")))
    produtos = driver.find_elements(By.CLASS_NAME, "poly-card__content")
    links = []
    for p in produtos:
        try:
            link = p.find_element(By.CLASS_NAME, "poly-component__title").get_attribute("href")
            if link: links.append(link)
        except: pass
        break
    return links

def extrair_info_produto(driver, link):
    driver.get(link)
    WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.CSS_SELECTOR, ".ui-pdp-title")))
    nome = driver.find_element(By.CSS_SELECTOR, ".ui-pdp-title").text
    preco_ori = driver.find_element(By.CSS_SELECTOR, ".andes-money-amount--previous").text.replace("\n", "")[2:]
    preco_desc = driver.find_element(By.CSS_SELECTOR, ".ui-pdp-price__second-line .andes-money-amount--compact").text.replace("\n", "")[2:]
    try:
        desconto = driver.find_element(By.CSS_SELECTOR, ".andes-money-amount__discount").text
    except:
        desconto = "Sem desconto"
    return {"nome_produto": nome, "link": link, "preco_original": preco_ori, "preco_desconto": preco_desc, "desconto_percentual": desconto}

def extrair_link_afiliado(driver):
    try:
        driver.find_element(By.ID, "P0-2").click()
        time.sleep(3)
        return driver.find_element(By.CSS_SELECTOR, ".andes-form-control__field--multiline").get_attribute("value")
    except:
        return None

def gerar_prompt_para_meta_ai(nome_produto):
    return f"gere uma imagem publicitária realista e comercial de alta qualidade para redes sociais, que represente claramente o produto que esta sendo mensionado em: {nome_produto}. Foque na função e aparência real dele, mas não coloque imagem de pessoas."

def enviar_produtos_para_meta_e_grupo(driver, mensagens_info, grupo):
    try:
        driver.get("https://web.whatsapp.com/")
        WebDriverWait(driver, 20).until(EC.presence_of_element_located((By.XPATH, '//div[@contenteditable="true"][@data-tab="3"]')))
        time.sleep(20)  # <-- AGUARDA 1 MINUTO APÓS ABRIR O WHATSAPP

        for mensagem, nome_produto in mensagens_info:
            campo_pesquisa = driver.find_element(By.XPATH, '//div[@contenteditable="true"][@data-tab="3"]')
            campo_pesquisa.clear()
            campo_pesquisa.send_keys("Meta AI")
            time.sleep(2)
            WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.XPATH, "//span[@title='Meta AI']"))).click()
            time.sleep(2)

            campo_msg = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.XPATH, '//div[@contenteditable="true"][@data-tab="10"]')))
            campo_msg.send_keys(gerar_prompt_para_meta_ai(nome_produto))
            campo_msg.send_keys(Keys.ENTER)
            print(f"🎨 Gerando imagem de: {nome_produto}")
            time.sleep(20)

            encaminhar_botoes = WebDriverWait(driver, 15).until(
                EC.presence_of_all_elements_located((By.XPATH, '//div[@role="button" and @aria-label="Encaminhar mídia"]')))
            encaminhar_botoes[-1].click()
            time.sleep(2)

            campo_busca = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.XPATH, '//div[@contenteditable="true" and @role="textbox"]')))
            campo_busca.click()
            campo_busca.send_keys(grupo)
            time.sleep(2)
            WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.XPATH, f"//span[@title='{grupo}']"))).click()
            time.sleep(2)

            campo_mensagem = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.XPATH, '//div[@aria-label="Adicione uma mensagem..."][@role="textbox"]')))
            for linha in mensagem.strip().split("\n"):
                campo_mensagem.send_keys(linha.replace("*", ""))
                campo_mensagem.send_keys(Keys.SHIFT + Keys.ENTER)
            time.sleep(1)

            botao_enviar = WebDriverWait(driver, 10).until(EC.element_to_be_clickable((By.XPATH, '//div[@role="button"]//span[@data-icon="wds-ic-send-filled"]')))
            botao_enviar.click()
            print("📨 Mensagem e imagem enviadas.")
            time.sleep(3)

    except Exception as e:
        print(f"❌ Erro durante envio para grupo: {e}")

def main():
    print("🚀 Iniciando...")
    iniciar_chrome()
    driver = iniciar_driver()
    sheet = conectar_planilha("Mercado livre")
    links = coletar_links_da_pagina(driver, "https://www.mercadolivre.com.br/ofertas")
    print(f"🔗 {len(links)} produtos coletados")

    mensagens_info = []

    for i, link in enumerate(links):
        print(f"\n🔍 Produto {i+1}/{len(links)}")
        info = extrair_info_produto(driver, link)
        if info:
            afiliado = extrair_link_afiliado(driver)
            if afiliado:
                info["link_afiliado"] = afiliado
            enviar_para_sheets(sheet, info)

            texto_base = f"{info['nome_produto']}\nDe R$ {info['preco_original']}\nAgora por R$ {info['preco_desconto']}\nDesconto: {info['desconto_percentual']}\n{info.get('link_afiliado', info['link'])}"
            texto_final = reescrever_com_ia(texto_base)
            mensagens_info.append((texto_final, info['nome_produto']))

    enviar_produtos_para_meta_e_grupo(driver, mensagens_info, grupo="Lá em casa - familia")
    input("⏹ Pressione Enter para sair...")
    driver.quit()

if __name__ == "__main__":
    main()
