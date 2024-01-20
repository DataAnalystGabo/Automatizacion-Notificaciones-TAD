# Automatización de tarea con Python y Selenium

### Introducción

En este repositorio, encontrarán una solución para un desafío: el envío masivo de notificaciones electrónicas a través de la plataforma oficial del Gobierno de la Ciudad de Buenos Aires a múltiples destinatarios utilizando Python y la librería de webscraping Selenium.

### Contexto

Durante mi paso en la Gerencia Operativa de Reclamos en el Ministerio de Espacio Público e Higiene Urbana, surgió la necesidad de notificar electrónicamente a través de la plataforma oficial del Gobierno de la Ciudad de Buenos Aires a 13 destinatarios sobre diversas incidencias encontradas en el espacio público. La tarea era sencilla y monótona; el cuerpo de las notificaciones siempre respetaba la misma estructura, los destinatarios eran siempre los mismos 13 correos electrónicos y los archivos adjuntos se subían de la misma forma. Lo único que variaba en la tarea era la calle y altura en donde se relevó cada incidencia que iban tanto en el encabezado del texto como en su cuerpo.

La base de datos que contenía la información acumulaba más de 3000 filas o registros. Aunque la tarea era relativamente fácil de ejecutar, implicaba que los miembros de la gerencia dejaran sus tareas habituales para abocarse a esta otra función. Fue entonces cuando, junto al Gerente del área, nos preguntamos: ¿existirá una forma de automatizar esta tarea y permitir que el equipo continúe cumpliendo con sus roles actuales?

### Filosofía del proyecto

El objetivo de este repositorio es demostrar mi capacidad de resolver un problema en poco tiempo y con recursos limitados. Quizás exista una forma más eficiente de hacer esto mismo, pero dejame decirte que esto funcionó y cumplió con su tarea. Muchas veces lo ideal no va de la mano con la realidad. En ocasiones, nos encontramos con el reloj en nuestra contra, con pocos recursos humanos, técnicos o presupuestarios y es en esos momentos donde verdaderamente ponemos a prueba nuestro ingenio y creatividad.

Espero que algo de lo que está aquí te sirva.

Happy Coding!

--------------------------------------------------------------------------------------------------------------------

### Herramientas

| Herramienta | Documentación |
| ------ | ------ |
| Python | https://docs.python.org/3/ |
| Selenium | https://www.selenium.dev/documentation/ |
|  Tqdm | https://tqdm.github.io/ |
| Jupyter Lab | https://docs.jupyter.org/en/latest/ |
| Git | https://git-scm.com/doc |
|  GitHub | https://github.com/features |


### Features

- Ingresar a la plataforma de notificaciones electrónicas del Gobierno de la Ciudad de Buenos Aires.
- Administrar claves, usuarios, correos electrónicos.
- Importar la base de datos en formato csv.
- Importar las imágenes de las incidencias previamente preparadas en formato pdf.
- Completar de manera automática todos los campos requeridos para el envío de las notificaciones electrónicas según las normas.

### Arquitectura de archivos

| Herramienta | Documentación |
| ------ | ------ |
| Resources | Carpeta que contiene los recursos tales como la base de datos y la base de imágenes. |
| config.py | Archivo ignorado por Git que contiene claves de acceso, usuarios, correos electrónicos. |
| index.ipynb | Archivo que contiene el código procedural escrito en Python. |

### Paradigma de programación

Es importante destacar que este código está enfocado en realizar una tarea específica (automatizar el proceso de notificaciones electrónicas), y el estilo de programación elegido (procedural) es apropiado para la naturaleza de la tarea. Este estilo facilita la comprensión del flujo del programa y la realización de tareas secuenciales.

--------------------------------------------------------------------------------------------------------------------
### Importación

En esta celda importamos las librerias a utilizar: 
- csv par leer la base de datos
- selenium para el manejo del navegador web
- request y config para la importación de claves, usuarios y correos electrónicos
- tqdm nos brinda una interface para monitorear el avance de envio de notificaciones electrónicas
- time para el control de los tiempos de espera entre cada ejecución del código

```sh
import csv
import selenium
import requests
from config import *
from tqdm import tqdm
from time import sleep
from selenium import webdriver
from pywinauto.keyboard import send_keys
from pywinauto.application import Application
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
```

### new_notification()

Función diseñada que selecciona la opción de una nueva notificación electrónica dentro de la plataforma.

```sh
def new_notification():
    try:
        XPATH_NOTIFICACTION_ELECTR = '//li[@title="Notificaciones Electronicas"]'
        XPATH_NEW_NOTIFICATION     = '//li[@title="Nueva Notificación"]'
        
        notification_electr        = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.XPATH, XPATH_NOTIFICACTION_ELECTR)))
        notification_electr.click()
        
        new_notification           = WebDriverWait(driver, 30).until(EC.presence_of_element_located((By.XPATH, XPATH_NEW_NOTIFICATION)))
        new_notification.click()
    except Exception as e:
         print(f'Ocurrió un error al intentar generar una nueva notificación {e}')
```

### add_companies()

Esta función cumple la tarea de cargar los cuit de las 13 destinatarios, los cuales serán notificados.
```sh
def add_companies(companies):
    
    XPATH_INPUT_CUIT    = '//input[@name="cuit1"]'
    XPATH_BUTTONS       = '//td[@class="z-row-inner"]//button[@class="z-button"]'
    
    for razon, cuit in companies.items():
        try: 
            input_cuit    = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.XPATH, XPATH_INPUT_CUIT)))
            input_cuit.send_keys(cuit)
            
            buttons       = WebDriverWait(driver, 10).until(EC.presence_of_all_elements_located((By.XPATH, XPATH_BUTTONS)))
            button_search = buttons[0]
            button_search.click()

            #Si encontró la empresa, avanzará:
            WebDriverWait(driver,10).until(EC.visibility_of_element_located((By.XPATH, f'//span[@class="z-label"][contains(text(), "{razon}")]')))
            
            button_add    = buttons[1]
            button_add.click()

            #Si se cargó la empresa, avanzará:
            WebDriverWait(driver,10).until(EC.visibility_of_element_located((By.XPATH, f'//div[@class="z-listcell-content"][contains(text(), "{razon}")]')))
                
        except Exception as e:
            print(f"Ocurrió un error al intentar agregar la empresa con CUIT {cuit}: {e}")
```

### add_pdf()

Aquí, cargamos los pdfs por cada registro. Estos archivos contienen las imágenes que describen la incidencia encontrada en la acera.

```sh
def add_pdf(num_sap):
    input_upload_file =  driver.find_elements(By.XPATH, '//span[@class="z-radiogroup"]//span[@class="z-radio"]')[3]
    input_upload_file.click()
    sleep(1)
    
    button_add_file = driver.find_elements(By.XPATH, './/i[@class="glyphicon glyphicon-plus"]')[6]
    button_add_file.click()
    sleep(1)
    
    app = Application().connect(title_re='Abrir')
    path_pdf = fr'C:\Users\Usuario\OneDrive\Escritorio\DGFU\010 - Automatizacion Notificaciones TAD\resources\images\{num_sap}'
    app.Abrir.Edit.set_text(path_pdf)
    app.Abrir.Edit.type_keys('{ENTER}')
```


### send_notification()

Esta función cumple la tarea de enviar la notificación electrónica una vez que se hayan cargado y completado todos los campos dentro de la plataforma.

```sh
def send_notification(user):
    #Boton para seleccionar la opción "Quiero recibir un aviso cuando se firme la tarea"
    XPATH_BUTTON_IWANT   = '//div[@class="z-window-content"]//div[@class="z-hlayout"]//div[@class="z-hlayout-inner"]//input[@type="checkbox"]'
    XPATH_BUTTON_SEND    = '//div[@class="z-hlayout"]//div[@class="z-hlayout-inner"]//button[@class="z-button"]'
    XPATH_INPUT_USER     = './/input[@placeholder="Ingrese el usuario"]'
    XPATH_BUTTON_SEND_OK = './/tbody[@class="z-rows"]//tr[@class="GridLayoutNoBorder z-row"]//td[@class="z-row-inner"]//div[@class="z-row-content"]//button[@class="z-button"]'
    XPATH_BUTTON_NEXT    = './/button[@class="z-messagebox-button z-button"]'
    
    input_aviso = WebDriverWait(driver, 10).until(EC.presence_of_all_elements_located((By.XPATH, XPATH_BUTTON_IWANT)))
    input_aviso = input_aviso[1]
    input_aviso.click()

    button_send = driver.find_elements(By.XPATH, XPATH_BUTTON_SEND)[9]
    button_send.click()

    input_send_username = WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.XPATH, XPATH_INPUT_USER)))
    input_send_username.send_keys(user)

    button_send_notif = driver.find_element(By.XPATH, XPATH_BUTTON_SEND_OK)
    button_send_notif.click()

    button_next =  WebDriverWait(driver, 10).until(EC.presence_of_element_located((By.XPATH, XPATH_BUTTON_NEXT)))
    button_next.click()
```

### app()
Por último, la función app() nuclea y organiza todas las demás funciones y cumple con el objetivo de ejecutar la tarea.

```sh
#Funcion de envio de notificación por cada posición

def app(csv_path):

    #Ingreso a GEDO ------------------------------
    login(USERNAME, PASSWORD)
    
    #Nueva notificación---------------------------
    new_notification()


    
    with open(csv_path) as csv_file:
        reader = csv.reader(csv_file, delimiter=';')
        rows   = list(reader)
        
        for row in tqdm(rows, desc="Enviando notificaciones"):

            response = requests.get(URL)
 
            try:
                if response.status_code == 200:
                
                    num_sap  = row[1]
                    position = row[2]
        
                    add_companies(COMPANIES)
                    #sleep(0.5)
                
                    #Redactando el título y el cuerpo de la notificación----------------------------------------------------------------------------------------------------------------------------------------------
                    XPATH_INPUT_TITLE  = '//div[@class="z-hlayout-inner"]//input[@class="z-textbox"]'
                    XPATH_INPUT_BODY   = '//textarea[@placeholder="Ingresar texto de la notificación"]'
                    XPATH_BUTTON_CHECK = '//tbody[@class="z-rows"]//tr[@class="z-row"]//td[@class="z-row-inner"]//div[@class="z-row-content"]//div[@class="z-hlayout"]//div[@class="z-hlayout-inner"]//button[@class="z-button"]'
                    
                    
                    input_referencia = driver.find_element(By.XPATH, XPATH_INPUT_TITLE)
                    input_referencia.send_keys(f"S/Faltante {position}")
                    
                    input_cuerpo     = driver.find_element(By.XPATH, XPATH_INPUT_BODY)
                    input_cuerpo.send_keys(f"Por medio de la presente se lo intima a que en el plazo de 72 horas hábiles, desde recibida esta intimación, verifique e informe, conforme Articulo 5 de la Ley 4760, si la anomalía detectada sobre {position}, correspondiente a una cámara con tapa faltante sobre la acera), pertenece a la infraestructura de su empresa. En caso de corresponder, se lo intima en un plazo de 5 días hábiles a proceder a la reposición de tapa y/o anulación de la misma según corresponda, bajo el apercibimiento de aplicar el régimen sancionatorio previstas en la materia en el Régimen de Faltas de la Ciudad Autónoma de Buenos Aires (Ley N° 451). \n\n Asimismo, se lo intima a que en igual plazo presente formal descargo con muestras fotográficas en las que se verifique el cumplimiento de lo intimado en el primer párrafo de esta notificación ante la oficina de la Dirección General sita en Av. Martin García 346, 3°piso, en el horario de 10 a 15 hs o, mediante el correo electrónico de la Mesa de Entradas virtual {email usuario}, todo ello, bajo apercibimiento de proceder al labrado de las correspondientes actas de comprobación, remoción y anulación de las instalaciones descriptas como irregulares. \n\n Se hace constar que el riesgo a terceros derivado de las instalaciones en deficiente estado habilita a esta dirección general a proceder cautelarmente respecto de las mismas, tomando la intervención física necesaria.")
                    
                    button_check     = driver.find_elements(By.XPATH, XPATH_BUTTON_CHECK)[0]
                    button_check.click()
        
                    #Adjuntando el pdf con las imágenes----------------------------------------------------------------------------------------------------------------------------------------------------------------
                    add_pdf(num_sap)
                    WebDriverWait(driver,20).until(EC.visibility_of_element_located((By.XPATH, f'//div[@class="z-listcell-content"][contains(text(), "{num_sap}")]')))
                            
                    #Función de envío----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
                    send_notification(USER)
                    
            except requests.exceptions_RequestException as e:
                print(f'Error de conexión: {e}')
```
