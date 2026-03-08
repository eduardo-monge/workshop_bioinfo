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

Agora que validamos o comando, vamos rodar para todos os arquivos que limpamos anteriormente. da forma como rodaríamos na vida real. 

⚠️ **Atenção:**
O arquivo SAM é muito pesado e ocupa muito espaço no computador, por isso normalmente não o geramos. Na prática, usamos `view` do [samtools](https://www.htslib.org/doc/samtools-depth.html) para converter automaticamente o SAM em BAM. 

```bash
# Loop para alinear y convertir todas las muestras Single-End
for file in ../trimmed/*_clean.fastq; do
    # Extraer solo el nombre de la muestra (ej: Acro_01)
    SAMPLE=$(basename $file _clean.fastq)
    
    echo "Alineando y convirtiendo la muestra: $SAMPLE"

    bwa mem -t 4 \
        -R "@RG\tID:$SAMPLE\tSM:$SAMPLE\tPL:ILLUMINA" \
        ../../reference/acrocomia_ref.fasta \
        $file |\
 samtools view -b - > ${SAMPLE}_nofilers.bam
done
```

Vamos agora usar a mesma ferramenta de `view` para visualizar o resultado do bam.

```bash
samtools view ${SAMPLE}_nofilers.bam | head
```

❓O que é cada parte do arquivo bam?

# 4. Visualizações e filtros pós-alinhamento
Agora já temos nossas sequências alinhadas no formato BAM. No entanto, antes de poder usá-las para chamar os SNPs, precisamos verificar a qualidade do mapeamento e realizar os filtros necessários.

Vamos aplicar um **Filtro de Qualidade de Mapeamento (MAPQ)** e, em seguida, organizaremos as informações.

### A. Filtrar e ordenar (Samtools)
Os programas posteriores precisam que as leituras estejam ordenadas da esquerda para a direita ao longo dos cromossomos (coordenadas genômicas). Podemos filtrar as leituras ruins e ordená-las ao mesmo tempo usando nosso velho amigo, o Pipe (`|`).

```bash
# Filtrar reads não mapeados (-F 4), reter reads com qualidade >= 20 e ordenar por coordenadas
samtools view -F 4 -q 20 -b Acro_01.bam |\
samtools sort -o Acro_01_filter.sorted.bam

# Indexar o arquivo ordenado (Gera o índice posicional .bai)
samtools index Acro_01.sorted.bam
```

### B. Mitigação de Vieses Técnicos: Duplicatas de PCR (Picard)
Durante a confecção das bibliotecas genômicas, a etapa de amplificação por PCR pode introduzir um viés estocástico, gerando cópias artificiais (clones) de um mesmo fragmento de DNA. Se estas duplicatas não forem expurgadas, o algoritmo de Variant Calling interpretará erroneamente essas cópias como evidência de alta profundidade de cobertura para um alelo específico, distorcendo severamente as estimativas de frequências alélicas da população. 

Usaremos a ferramenta `MarkDuplicates` de [Picard](https://gatk.broadinstitute.org/hc/en-us/articles/360037052812-MarkDuplicates-Picard) para mitigar este artefato.

```bash
# Executar a remoção de duplicatas com Picard MarkDuplicates
picard MarkDuplicates \
    I=Acro_01_filter.sorted.bam \
    O=Acro_01.dedup.bam \
    M=Acro_01_dup_metrics.txt \
    REMOVE_DUPLICATES=true 

# A geração de um novo arquivo BAM exige uma nova indexação
samtools index Acro_01.dedup.bam

# -----------------------------------------
Os parámetros que estamos usando:
#I=Input
#O=Output
#M= Metrics. Gera um relatório quantitativo contendo a proporção exata de sequências que consistiam em artefatos de PCR.
#REMOVE_DUPLICATES=true. Fala para o programa deletar fisicamente as duplicatas dos dados, não apenas marcá-las.
# -----------------------------------------
```

### 🤖 Automação do Pós-Processamento
Tendo validado a eficácia do protocolo em uma amostra empírica, implementaremos um laço de repetição iterativo (loop) para processar toda a coorte de forma automatizada.
```bash
# Certifique-se de que o diretório ativo seja a pasta de alinhamento
cd ~/workshop_bioinfo/data/processed/alignment

for bam in *.bam; do
    SAMPLE=$(basename $bam .bam)
    echo "Iniciando processamento da amostra: $SAMPLE"

    # 1. Filtragem (-F 4 e -q 20) acoplada à ordenação posicional
    echo "Aplicando filtros (MAPQ/Flags) e Ordenando"
    samtools view -F 4 -q 20 -b $bam | samtools sort -o ${SAMPLE}.sorted.bam
    
    # 2. Supressão de Duplicatas de PCR
    echo "Identificando e removendo duplicatas técnicas"
    picard MarkDuplicates \
        I=${SAMPLE}.sorted.bam \
        O=${SAMPLE}.dedup.bam \
        M=${SAMPLE}_dup_metrics.txt \
        REMOVE_DUPLICATES=true 

    # 3. Indexação do BAM final
    echo "Computando índice genômico"
    samtools index ${SAMPLE}.dedup.bam

    # 4. Higiene de Dados: Exclusão de matrizes intermediárias
    rm $bam
    rm ${SAMPLE}.sorted.bam
done
```
⚠️ **Atenção:**
A exclusão de arquivos intermediários (.bam bruto e .sorted.bam) previne a exaustão da capacidade de armazenamento do servidor, retendo estritamente o arquivo .dedup.bam, que está lapidado e pronto para a inferência de variantes.


# 5. Check as estatísticas básicas do bam
O último passo é revisar as estatísticas finais do mapeamento para verificar se tudo está correto e se aproveitamos bem as sequências. Este é um passo vital, pois nos permite considerar se precisamos ajustar os parâmetros de mapeamento ou se podemos seguir em frente. 

Para esta fase, utilizaremos duas ferramentas do samtools: 
1. `flagstat` para gerar estatísticas rápidas de alinhamentos do arquivo bam.
2. `depth` para calcular a cobertura do nosso arquivo bam.

```bash
# 1. Navegar para o diretório com os BAM finais
cd ~/workshop_bioinfo/data/processed/alignment

# 2. Criar pasta para relatórios de QC
mkdir stats

# 3. Criar arquivo de resumo da profundidade média
printf "Amostra\tProfundidade_Media\n" > ../stats/resumo_profundidade.txt

# 4. Loop para todas as amostras
for bam in *.dedup.bam; do

    SAMPLE=$(basename "$bam" .dedup.bam)
    echo "Processando amostra: $SAMPLE"

    # A. Estatísticas de alinhamento
    samtools flagstat -a "$bam" > ../qc_alignment/"${SAMPLE}_flagstat.txt"

    # B. Cálculo da profundidade média
    DEPTH=$(samtools depth "$bam" | awk '{sum+=$3} END {print sum/NR}')

    # Guardar resultado no resumo
    printf "%s\t%.2f\n" "$SAMPLE" "$DEPTH" >> ../qc_alignment/resumo_profundidade.txt

done

# -----------------------------------------
Os parámetros que estamos usando:
#printf=escrever o cabeçalho da tabela, contendo duas colunas:
# -Amostra — identificação da amostra analisada
# -Profundidade_Media — valor médio da cobertura de sequenciamento
# O símbolo > indica redirecionamento de saída, o que significa que o conteúdo será escrito no arquivo especificado. Caso o arquivo já exista, ele será sobrescrito, garantindo que um novo resumo seja gerado para cada execução do script.
#samtools flagstat -a=  Calcula um conjunto de estatísticas fundamentais do alinhamento. Incluindo -a incluimos posições com cobertura 0. 
#samtools depth = calcula a profundidade média de cobertura genômica para cada amostra.
# -----------------------------------------

```
Agora abra os arquivos gerados e veja as estatísticas gerais. 

❓Como foi o nosso mapeamento?

---
# Mapeando no dia a dia
Como vimos, durante esse processo são gerados muitos arquivos intermediários que não usamos para a chamada de SNPs. Isso ocupa espaço em nosso computador/servidor. 

Para mitigar esse problema, na vida real usamos um único script que faz todo o mapeamento, conversão e filtragem dos arquivos e retorna no final apenas um arquivo .bam final pronto para a chamada de SNPs. 

Quando estamos começando com a análise de dados, é recomendável seguir o processo passo a passo para entender o que está sendo feito e ter certeza do processo. Quando já estivermos familiarizados com os programas e parâmetros, podemos otimizar essa etapa. 

💡 Script "Tudo-em-Um"
```bash
# 1. Indexar o genoma de referência (Isso é feito apenas UMA vez antes do loop)
echo "Indexando o genoma de referência..."
bwa index ../../reference/acrocomia_ref.fasta

# 2. O Lloop de processamento
for file in ../trimmed/*_clean.fastq; do
    # Extrair o nome da amostra
    SAMPLE=$(basename $file _clean.fastq)
    echo "Iniciando processamento em cadeia da amostra: $SAMPLE"

    # PASSO A: Alinhamento (BWA) + Conversão/Filtro (View) + Ordenação (Sort)
    # Note os pipes (|) conectando os programas sem gerar arquivos intermediários pesados
    bwa mem -t 4 -R "@RG\tID:$SAMPLE\tSM:$SAMPLE\tPL:ILLUMINA" \
        ../../reference/acrocomia_ref.fasta $file | \
    samtools view -F 4 -q 20 -b - | \
    samtools sort -o ${SAMPLE}.tmp.sorted.bam

    # PASSO B: Remoção de Duplicatas (Picard)
    picard MarkDuplicates \
        I=${SAMPLE}.tmp.sorted.bam \
        O=${SAMPLE}.final.bam \
        M=${SAMPLE}_dup_metrics.txt \
        REMOVE_DUPLICATES=true \
        VALIDATION_STRINGENCY=SILENT

    # PASSO C: Indexação do BAM Final
    samtools index ${SAMPLE}.final.bam

    # PASSO D: Limpeza inteligente (Apagar o arquivo temporário ordenado)
    rm ${SAMPLE}.tmp.sorted.bam
    
    echo "Amostra $SAMPLE finalizada com sucesso!"
done
```

