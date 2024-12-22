# Instalar dependências necessárias no Google Colab
!pip install requests
!pip install beautifulsoup4
!pip install pandas

# Importação das bibliotecas necessárias
import requests
from bs4 import BeautifulSoup
import pandas as pd
import re
import time

# Função para buscar vagas de emprego em um site específico
def buscar_vagas_em_site(url, parametros=None):
    headers = {
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36'
    }

    try:
        response = requests.get(url, headers=headers, params=parametros)

        # Verificação de erros HTTP
        if response.status_code == 403:
            print(f"Erro 403: Acesso proibido ao site {url}.")
            return []
        elif response.status_code == 429:
            print(f"Erro 429: Muitas requisições para o site {url}, esperando 10 segundos para tentar novamente.")
            time.sleep(10)
            return buscar_vagas_em_site(url, parametros)  # Tentar novamente após espera

        response.raise_for_status()  # Verifica se a requisição foi bem sucedida
        soup = BeautifulSoup(response.text, 'html.parser')

        vagas = []
        for vaga in soup.find_all('a', class_='jobtitle'):
            titulo = vaga.get_text()
            link = vaga.get('href')

            # Verificar se a vaga é remota
            if "remote" in titulo.lower() or "telecommute" in titulo.lower() or "work from home" in titulo.lower():
                # Tenta encontrar o salário na vaga (em EUR, USD ou CHF)
                salario = encontrar_salario(soup)

                if salario and salario >= 10000:
                    vagas.append({'Título': titulo, 'Link': link, 'Salário': salario})

        return vagas
    except Exception as e:
        print(f"Erro ao buscar vagas no site {url}: {e}")
        return []

# Função para tentar encontrar o salário nas vagas
def encontrar_salario(soup):
    # Regexp para buscar valores de salários em EUR, USD, e CHF
    salario_eur = re.search(r'(\d{1,3}(?:[.,]\d{3})*(?:[.,]\d+)?)(?=\s*€)', soup.get_text())
    salario_usd = re.search(r'(\d{1,3}(?:[.,]\d{3})*(?:[.,]\d+)?)(?=\s*\$)', soup.get_text())
    salario_chf = re.search(r'(\d{1,3}(?:[.,]\d{3})*(?:[.,]\d+)?)(?=\s*CHF)', soup.get_text())

    # Verifica se encontrou algum salário e retorna o maior valor encontrado
    salarios = []
    if salario_eur:
        salarios.append(float(salario_eur.group(1).replace(',', '').replace('€', '').strip()))
    if salario_usd:
        salarios.append(float(salario_usd.group(1).replace(',', '').replace('$', '').strip()))
    if salario_chf:
        salarios.append(float(salario_chf.group(1).replace(',', '').replace('CHF', '').strip()))

    if salarios:
        return max(salarios)
    else:
        return None

# Exemplo de URLs de sites de vagas (agora mais sites)
urls_vagas = [
    'https://www.indeed.com/q-remote-jobs.html',  # Indeed
    'https://www.linkedin.com/jobs/search?keywords=remote',  # LinkedIn
    'https://www.glassdoor.com/Job/remote-jobs-SRCH_KO0,6.htm',  # Glassdoor
    'https://weworkremotely.com/',  # We Work Remotely
    'https://remote.co/remote-jobs/',  # Remote.co
    'https://jobspresso.co/',  # Jobspresso
    'https://angel.co/jobs',  # AngelList
    'https://remotive.io/',  # Remotive
    'https://remoteok.io/',  # Remote OK
    'https://www.flexjobs.com/',  # FlexJobs
]

# Coletar as vagas de cada site
todas_vagas = []
for url in urls_vagas:
    vagas_site = buscar_vagas_em_site(url)
    todas_vagas.extend(vagas_site)

# Exemplo de uma tabela com vagas encontradas
df_vagas = pd.DataFrame(todas_vagas)

# Verifique as vagas encontradas
print(f"Vagas encontradas: {df_vagas.shape[0]} vagas")
print(df_vagas.head())

# Competências mais demandadas - Exemplo simples
competencias_demandadas = [
    'Python', 'SQL', 'Machine Learning', 'AWS', 'DevOps', 'Docker', 'JavaScript', 'Data Analysis', 'Cloud', 'Project Management'
]

# Simulação de cursos online relacionados às competências
cursos_online = {
    "Competência": competencias_demandadas,
    "Curso": [
        "Python Avançado", "SQL para Análise de Dados", "Introdução ao Machine Learning", "AWS Essentials",
        "Fundamentos de DevOps", "Docker para Iniciantes", "JavaScript Básico", "Análise de Dados com Python",
        "Fundamentos de Cloud Computing", "Gestão de Projetos Ágeis"
    ],
    "Plataforma": [
        "Coursera", "edX", "Udemy", "Coursera", "LinkedIn Learning", "Udemy", "LinkedIn Learning", "Coursera",
        "AWS Training", "Coursera"
    ],
    "Link para Curso": [
        "https://www.coursera.org/courses/python", "https://www.edx.org/course/sql-data-analysis", "https://www.udemy.com/course/machine-learning",
        "https://aws.amazon.com/training", "https://www.linkedin.com/learning/devops-basics", "https://www.udemy.com/course/docker",
        "https://www.linkedin.com/learning/javascript-for-beginners", "https://www.coursera.org/courses/data-analysis-python",
        "https://aws.amazon.com/training/cloud", "https://www.coursera.org/courses/project-management"
    ]
}

# Criando o DataFrame de cursos online
df_cursos = pd.DataFrame(cursos_online)

# Exibir as vagas e cursos recomendados
print("\nVagas Remotas Encontradas com Salários Acima de 10.000 EUR/USD/CHF:")
print(df_vagas.head())

print("\nCursos Recomendados por Competência:")
print(df_cursos)

# Salvar os resultados em CSVs locais
df_vagas.to_csv("/content/vagas_remotas_encontradas.csv", index=False)
df_cursos.to_csv("/content/cursos_recomendados.csv", index=False)

print("\nArquivos CSV salvos: 'vagas_remotas_encontradas.csv' e 'cursos_recomendados.csv'")
