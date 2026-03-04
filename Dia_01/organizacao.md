# 💻 Prática Dia 1: Organização de Projeto e Exploração de Dados

Nesta primeira prática, vamos configurar nosso ambiente de trabalho e aprender comandos essenciais para manipular e explorar arquivos de sequenciamento.

---

## 1. Configuração da Estrutura de Diretórios

A organização é o primeiro passo para a reprodutibilidade. Vamos criar uma hierarquia lógica para o nosso projeto de bioinformática.

```bash
# Navegar para o diretório pessoal
cd ~

# Criar a pasta raiz do curso (caso ainda não tenha criado)
mkdir -p ~/workshop_bioinfo && cd ~/workshop_bioinfo

# Criar subpastas estruturadas
mkdir -p data/raw data/processed/trimmed data/processed/aligned
mkdir -p results/qc results/vcf_raw results/vcf_filtered
mkdir -p scripts logs docs

# Verificar a estrutura criada
ls -R
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
