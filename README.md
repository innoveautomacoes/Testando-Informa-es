import os
import re
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
import base64

    # Função para extrair o valor do Cod
def extract_cod(html):
    match = re.search(r'<span>Cod: (\d+)</span>', html)
    return match.group(1) if match else 'NI'

    # Função para extrair URLs das imagens usando Selenium e salvar como base64
def extract_and_save_images_as_base64(html_content, output_folder, max_images=3):
    # Criar um arquivo HTML temporário
    temp_html_file = 'temp.html'
    with open(temp_html_file, 'w', encoding='utf-8') as file:
        file.write(html_content)

    # Configurar o Selenium WebDriver
    chrome_options = Options()
    chrome_options.add_argument('--headless')
    chrome_options.add_argument('--disable-gpu')
    chrome_options.add_argument('--no-sandbox')
    chrome_options.add_argument('--disable-extensions')
    chrome_options.add_argument('--disable-dev-shm-usage')
    service = Service('C:\\Users\\jozen\\Downloads\\chromedriver-win64\\chromedriver-win64\\chromedriver.exe')
    driver = webdriver.Chrome(service=service, options=chrome_options)

    try:
        # Abrir o arquivo HTML no navegador
        driver.get(f'file:///{os.path.abspath(temp_html_file)}')

        # Esperar as imagens carregarem
        WebDriverWait(driver, 10).until(EC.presence_of_all_elements_located((By.TAG_NAME, 'img')))

        # Encontrar e salvar as imagens
        img_elements = driver.find_elements(By.TAG_NAME, 'img')
        for idx, img in enumerate(img_elements[:max_images]):
            img_base64 = driver.execute_script("return arguments[0].src;", img)
            if img_base64.startswith('data:image'):
                img_base64 = img_base64.split(',')[1]  # Remover o prefixo 'data:image/jpeg;base64,'
                img_data = base64.b64decode(img_base64)
                image_file_name = f"{os.path.basename(output_folder)}_image_{idx+1}.jpg"
                with open(os.path.join(output_folder, image_file_name), 'wb') as img_file:
                    img_file.write(img_data)
                print(f"Imagem {idx+1} salva na pasta {output_folder} como {image_file_name}")

    finally:
        driver.quit()
        os.remove(temp_html_file)

    # Caminhos das pastas
input_folder = r'C:\Users\jozen\OneDrive\Área de Trabalho\Automação PetLove\Arquivos txt'
output_base_folder = r'C:\Users\jozen\OneDrive\Área de Trabalho\Automação PetLove\Imagens PetLove'
os.makedirs(output_base_folder, exist_ok=True)

    # Processar cada arquivo de texto na pasta
for file_name in os.listdir(input_folder):
    if file_name.endswith('.txt'):
        file_path = os.path.join(input_folder, file_name)
        with open(file_path, 'r', encoding='utf-8') as file:
            html_content = file.read()
            cod = extract_cod(html_content)
            output_folder = os.path.join(output_base_folder, cod)
            os.makedirs(output_folder, exist_ok=True)
            
            extract_and_save_images_as_base64(html_content, output_folder)
