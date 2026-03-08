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
```

### B. Gerar o índice
# 2. Indexar a referência usando o BWA
bwa index acrocomia_ref.fasta
