# 💻 Prática Dia 1: Organização de Projeto e Exploração de Dados

Nesta primeira prática, vamos configurar nosso ambiente de trabalho e aprender comandos essenciais para manipular e explorar arquivos de sequenciamento.

---

## 1. Configuração da Estrutura de Diretórios

A organização é o primeiro passo para a reprodutibilidade. Vamos criar uma hierarquia lógica para o nosso projeto de bioinformática. Vamos a seguir a lógica recomendada por [Noble (2009) "A Quick Guide to Organizing Computational Biology Project"](https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1000424)

<img width="4134" height="2308" alt="Noble (2009)" src="https://github.com/user-attachments/assets/84830a55-885a-4b5f-8e4e-014de6ded742" />

```bash
# Navegar para o diretório pessoal
cd ~

# Criar a pasta raiz do curso (caso ainda não tenha criado)
mkdir -p ~/workshop_bioinfo
cd ~/workshop_bioinfo

#Se quiser fazer tudo numa linha só
mkdir -p ~/workshop_bioinfo && cd ~/workshop_bioinfo

# -----------------------------------------
#A opção -p significa “pais”.
#Ela instrui o sistema a:
#1️⃣ Criar todos os diretórios pais, caso eles não existam
#2️⃣ Evitar gerar um erro se o diretório já existir
# -----------------------------------------

# Criar subpastas estruturadas
mkdir -p data/raw data/trimmed data/mapped
mkdir -p results/qc results/snp_calling results/vcf_filtered
mkdir -p scripts #Opcional para o tutorial, recomendado para seus projetos pessoais. 

# Verificar a estrutura criada
ls -R

# -----------------------------------------
#A opção -R significa “recursive”.
#Ele instrui o comando ls a listar o conteúdo dos diretórios e todos os seus subdiretórios recursivamente.
# -----------------------------------------
```
## 2. Download de Dados de Sequenciamento (FASTQ)
Vamos baixar um conjunto de dados reduzido (subset) para praticar os comandos de manipulação. Utilizaremos o comando wget.

```bash
# Entrar na pasta de dados brutos
cd data/raw

# Baixar arquivos FASTQ de exemplo
wget [https://raw.githubusercontent.com/josoga2/common-usage-files/master/sample_R1.fastq](https://raw.githubusercontent.com/josoga2/common-usage-files/master/sample_R1.fastq)
wget [https://raw.githubusercontent.com/josoga2/common-usage-files/master/sample_R2.fastq](https://raw.githubusercontent.com/josoga2/common-usage-files/master/sample_R2.fastq)

# Listar os arquivos e verificar o tamanho (legível para humanos)
ls -lh
```

💡 **NOTA DO DIA A DIA: Como recebo meus dados?**
> Na vida real, quando o sequenciamento das suas amostras finaliza, você não recebe os dados por pendrive. A empresa geralmente disponibiliza os arquivos em servidores FTP ou na nuvem oficial da [Illumina BaseSpace](https://www.illumina.com/products/by-type/informatics-products/basespace-sequence-hub.html). 
> 
> Nesses casos, você usaria o terminal para baixar os dados diretamente para o seu servidor. 
> 
> **Exemplos de comandos reais (Não execute agora):**
> ```bash
> # Opção A: Usando a ferramenta oficial da Illumina (BaseSpace CLI)
> # (Baixa o projeto inteiro de uma vez)
> bs download project --id 12345678 --output ./data/raw/
> 
> # Opção B: Usando wget com um link de servidor FTP seguro do laboratório
> # (O '-c' permite continuar o download de onde parou caso a internet caia)
> wget -c "ftp://usuario:senha@ftp.centro-sequenciamento.com/projeto/*.fastq.gz"
> ```


## 3. Exploração de Dados via Terminal
Arquivos de bioinformática costumam ser muito grandes. Aprender a "espiar" o conteúdo sem abrir o arquivo inteiro é fundamental.

A. Anatomia do FASTQ (Comando head e tail)
Cada read no formato FASTQ possui 4 linhas. Vamos ver o primeiro read do arquivo:
```bash
head -n 4 sample_R1.fastq
```

B. Contagem de Sequências
Podemos usar o contador de linhas (wc -l) para estimar o número de reads.
```bash
# Contar total de linhas
wc -l sample_R1.fastq
```

C. Busca com Grep
O comando grep é poderoso para encontrar padrões. Vamos verificar os cabeçalhos das sequências.
```bash
# Listar os primeiros 5 cabeçalhos (começam com @)
grep "^@" sample_R1.fastq | head -n 5
```

## 4. Gestão de Metadados (Prática com Nano)
Agora, vamos criar nossa tabela de metadados para vincular as amostras às suas informações biológicas.

```bash
# Voltar para a pasta data
cd ..

# Criar o arquivo de metadados
nano metadata.tsv
```
Dentro do editor Nano, digite o seguinte (use a tecla TAB para separar as colunas):

```bash
Sample_ID    Species    Location    Condition
sample_01    Acrocomia  Guatemala   Control
sample_02    Acrocomia  Guatemala   Tratamento
```
(Para salvar: Ctrl + O, depois Enter. Para sair: Ctrl + X)



💡 **Dica de Sobrevivência: Usando o Excel a seu favor**
> 
> Trabalhar com editores de texto dentro do terminal, como o `nano`, pode ser um desafio no início, especialmente para quem ainda não está acostumados com ambientes de programação. Não se preocupe! 
> 
> Uma alternativa muito prática é organizar sua tabela de metadados no Excel e, em seguida, simplesmente copiar as células e colar dentro do terminal. Se você estiver trabalhando com o WSL (*Windows Subsystem for Linux*), essa integração de "copiar e colar" entre o Windows e o Linux é direta, rápida e facilita muito a vida.
> 
> Só lembra que no terminal do Linux, os atalhos clássicos `Ctrl+C` e `Ctrl+V` **não funcionam** (inclusive, o `Ctrl+C` serve para cancelar e abortar comandos em andamento). Para colar o texto que você copiou do Excel, você precisará **clicar com o botão direito do mouse** dentro da tela do terminal.
> 

**⚠️ Regras de Ouro para Nomes (Arquivos e Metadados):**
> Para evitar que os programas de bioinformática apresentem erros misteriosos no futuro, siga sempre estas três regras na hora de preencher sua tabela:
> 
> 1. **Sem caracteres especiais:** Evite símbolos como `@`, `#`, `$`, `%`, `&`, etc.
> 2. **Sem acentuação ou cedilha:** Não utilize acentos (´, ^, ~, `) ou a letra 'ç' (ex: escreva *populacao* em vez de *população*).
> 3. **Sem espaços em branco:** O Linux entende o espaço como o fim de um comando. Para separar palavras, utilize sempre o sublinhado/underscore (`_`) (ex: `amostra_populacao_01`).
