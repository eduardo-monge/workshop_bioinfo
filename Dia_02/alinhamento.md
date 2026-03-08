# 💻 Prática Dia 2 (Tarde): Alinhamento e Mapeamento de Sequências
Agora que temos nossos dados limpos e filtrados (seja pelo fastp ou Trim Galore), o próximo passo é o Mapeamento. Vamos comparar cada read da nossa amostra com o Genoma de Referência para descobrir de qual parte do cromossomo aquela sequência veio.

---
## 1. Genoma de referência
Antes de começar, precisamos conhecer nosso genoma de referência e poder indexá-lo. 

### A. Baixe e conheça o genoma de referência 
O primeiro passo é obter o genoma de referência da Acrocomia aculeata (Acessão: GCA_055471735.1) directamente de NCBI. 

```bash
1. Criar pasta para a referência e entrar nela
cd ~/workshop_bioinfo/data
mkdir -p reference
cd reference
```
Ahora vamos a fazer o download do genoma via FTP (File Transfer Protocol), compactado diretamente do repositório do NCBI.

```bash
# 2. Baixar o genoma da Macaúba (Acrocomia aculeata)
wget -c https://ftp.ncbi.nlm.nih.gov/genomes/all/GCA/055/471/735/GCA_055471735.1_USP_Acracu_1.0/GCA_055471735.1_USP_Acracu_1.0_genomic.fna.gz

# 3. Descompactar o arquivo
gunzip GCA_055471735.1_USP_Acracu_1.0_genomic.fna.gz

# 4. Renomear para facilitar o uso (opcional, mas recomendado)
mv GCA_055471735.1_ASM5547173v1_genomic.fna acrocomia_ref.fasta
```
Agora vamos explorar o genoma de referência para nos familiarizarmos com ele. Lembre-se de que o genoma é um arquivo fasta. 
❓Como esperam que seja o genoma?
❓Quantas e quais sequências estão presentes no arquivo fasta que usaremos como referência?

```bash
head acrocomia_ref.fasta
grep -c "^>" acrocomia_ref.fasta
```
Uma boa prática é modificar o nome dos cromossomos para que seja muito mais simples ler nos próximos programas. Para isso, podemos usar o seguinte script para modificar e deixar o genoma pronto para fazer o índice. 

```bash
awk '
BEGIN {chr=1; un=1}
/^>/ {
    if ($0 ~ /chromosome/) {
        print ">CHR" chr
        chr++
    } else {
        print ">UNRE" un
        un++
    }
    next
}
{print}
' acrocomia_ref.fasta > acrocomia_ref_renamed.fasta

# -----------------------------------------
O que o script faz
#1. Detecta cabeçalhos FASTA (/^>/)
#2. Modifica apenas linhas que começam com >.
#3. Se o cabeçalho contiver “cromossomo” ($0 ~ /cromossomo/), então atribui:
#>CHR1
#>CHR2
#>CHR3
#usando o contador chr.
#4. Se não, presume que é um contíguo não resolvido:
#>UNRE1
#>UNRE2
#usando o contador un.
# -----------------------------------------
```

# 2. Indexar a referência usando o BWA
Com o genoma de referência pronto e conhecido, podemos gerar o índice. Lembre-se de que o índice é a estratégia para pesquisar rapidamente dentro do genoma e otimizar os métodos de mapeamento. 

Este é um processo muito simples, utilizando o programa [bwa](https://github.com/lh3/bwa).

```bash
bwa index acrocomia_ref_renamed.fasta
```

🔍 O que observar: 
Use o comando `ls -lh`. Você verá arquivos com extensões `.bwt`, `.ann`, `.pac`, `.sa` e `.amb`. 

Atenção: Se você mudar o arquivo .fasta de pasta, precisa levar todos esses arquivos de índice junto, senão o alinhamento falhará!


# 3. Mapeamento do genoma de referência utilizando o BWA-MEM
Agora vamos correr o [bwa mem](https://github.com/lh3/bwa) para mapear as reads da macaúba a esta referência. O alinhamento é o processo de encontrar a posição exata de cada read no genoma da Macaúba. Como nossas sequências são Single-End, o comando é direto. 

### A. Criando a estrutura de pastas
Primeiro, vamos organizar onde os resultados (arquivos SAM) serão salvos.

```bash
# 1. Criar pasta para o alinhamento e entrar nela
cd ~/workshop_bioinfo/data/processed
mkdir -p alignment
cd alignment
```

### B. Rodando o Alinhamento
Vamos testar o comando para a primeira amostra para compreender o que está sendo feito. 

```bash
Comando para alinhar UMA amostra
bwa mem -t 4 \
    -R "@RG\tID:Acro_01\tSM:Acro_01\tPL:ILLUMINA" \
    ../../reference/acrocomia_ref_renamed.fasta \
    ../trimmed/Acrocomia_pop1_.fastq \
    > Acro_01.sam

# -----------------------------------------
Os parámetros que estamos usando:
#bwa mem: Algoritmo de alinhamento BWA otimizado para leituras de 70–1000 bp.
#-t 4: Utiliza 4 threads (núcleos de CPU). Se o seu computador tiver mais núcleos, você pode aumentar (-t 8, -t 16)
#-R "@RG\tID:Acro_01\tSM:Acro_01\tPL:ILLUMINA": Grupo de leitura. Isso adiciona metadados ao BAM. Ferramentas como o GATK exigem isso.
#../../reference/acrocomia_ref_renamed.fasta: Seu genoma renomeado e indexado. Lembre-se de que todos os arquivos da indexação devem estar juntos. 
#../trimmed/Acrocomia_pop1_.fastq: As leituras limpas e sem adaptadores. 
#> Acro_01.sam: O símbolo de "maior que" redireciona a saída do programa. Salva o alinhamento no formato SAM.
# -----------------------------------------
```

Use o less para se mover dentro do arquivo SAM.
❓Como está o arquivo? O que significa da parte dele? 




