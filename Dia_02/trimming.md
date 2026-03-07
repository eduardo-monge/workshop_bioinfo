# 💻 Prática Dia 2: Demultiplexação, QC e Trimming

Nesta prática, vamos aprender a analisar as sequências para verificar a qualidade e fazer o trimming necessário. 

---
## 1. Demultiplexação com STACKS (`process_radtags`)
Quando sequenciamos bibliotecas de representação reduzida (como RADseq ou GBS), várias amostras são misturadas no mesmo tubo (multiplexadas). O primeiro passo é usar os "códigos de barras" (*barcodes*) para separar quem é quem. Geralmente, as empresas já fornecem os dados separados, mas também podemos fazê-lo nós mesmos. 

Existem diferentes programas e métodos, dependendo da biblioteca genômica que usamos. No caso do GBS/RAD-seq, um programa que pode ser utilizado é o módulo `process_radtags` do [STACKS](https://catchenlab.life.illinois.edu/stacks/comp/process_radtags.php).

### 0. Preparar os dados para o programa STACKS (El archivo de Barcodes)
Antes de rodar qualquer comando do STACKS para demultiplexar, o programa precisa de um "mapa" para saber como separar essa "sopa" de sequências. Para que o processo seja um sucesso, precisamos de três elementos fundamentais:

1. **A enzima de restrição:** Saber exatamente qual enzima (ou enzimas) foi usada na bancada do laboratório para cortar o DNA (ex: *EcoRI*, *MspI*, *PstI*).
2. **Os arquivos FASTQ brutos:** Que já baixamos na nossa pasta `data/raw`.
3. **O arquivo de Barcodes:** Um "mapa" de texto que vincula cada código de barras com o nome real da amostra.

### 📝 Como deve ser a planilha de amostras?

O STACKS é extremamente rigoroso com o formato deste arquivo. Ele deve ser um arquivo de texto simples (`.txt`), onde as colunas são separadas **exclusivamente pela tecla TAB** (tabulação). Não deve ter cabeçalhos (títulos de colunas) nem espaços em branco soltos.

A estrutura mais básica tem duas colunas: `[Sequência_do_Barcode]<tab>[Nome_da_Amostra]`

<img width="223" height="222" alt="image" src="https://github.com/user-attachments/assets/427204af-e5fb-4f8d-afae-a6930986df85" />

Os nomes das amostras podem ser os relevantes para o seu projeto. Lembre-se apenas das dicas para nomear amostras aprendidas no dia 1. 
Vamos criar nosso arquivo de *barcodes* diretamente no terminal:

```bash
# Navegar para a pasta principal de dados
cd ~/workshop_bioinfo/data

# Criar e abrir o arquivo com o editor nano
nano barcodes.txt

#Dentro do nano, você pode copiar e colar os códigos de barras e amostras de um arquivo Excel ou TXT do seu computador. Lembre-se de como fizemos no dia 1. 
```
(Para salvar: pressione Ctrl + O e depois Enter. Para sair: Ctrl + X)



### 1. Executar `process_radtags`


## 2. Demultiplexação com STACKS (`process_radtags`)
## 3. Demultiplexação com STACKS (`process_radtags`)
