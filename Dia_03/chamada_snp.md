# 💻 Prática Dia 3: Chamada de SNPs

Nesta prática, aprenderemos duas maneiras diferentes de chamar variantes, com foco na chamada de SNPs. Faremos isso considerando dois cenários diferentes: com e sem genoma de referência. 

---
## 0. Lembrando os tipos de variantes 
As variantes ou variaçoes genômicas são as posições em que as reads não são idênticas à referência. Assim identificaremos variações genética entre indivíduos de uma espécie ou entre espécies, o que fornecerá informações importantes sobre diversidade genética, saúde, evolução e muito mais.

Essas variantes podem ocorrer em uma única base ou em cromossomos inteiros. Entre os tipos mais importantes estão os SNPs, os indels e as variantes estruturais (SVs).

1. **SNPs (Single Nucleotide Polymorphisms):** É a substitução de uma única base no DNA por outra base. É o tipo de variante mais simple e frequente no genoma. Elas são marcadores altamente informativos, ya que podem ocorrer em regiões codificantes ou não codificantes, influenciando uma série de processos, genes e proteínas.
2. **INDELs (Inserções e Deleções):** Indels são variantes em que ocorrem inserções ou deleções de pequenas sequências de nucleotídeos em comparação com a referência. Elas geralmente envolvem poucos pares de bases (<50pb), mas podem causar um impacto funcional significativo — especialmente em regiões codificantes de genes.
3. **Variantes Estruturais (SVs):** São alterações genômicas em larga escala, geralmente envolvendo segmentos com dezenas a milhões de bases. Ao contrário dos anteriores, as SVs podem modificar significativamente a arquitetura genômica. Entre os principais tipos de SVs estão:
    + _Deleções_: remoção de segmentos inteiros do DNA
    + _Duplicações_: cópia adicional de uma região
    + _Inversões_: quando um trecho do DNA é invertido
    + _Translocações_: troca de segmentos entre cromossomos diferentes

![Types-of-Genetic-Variation-Single-nucleotide-variants-SNVs-and-indels-are-changes-that_W640](https://github.com/user-attachments/assets/a8a404bb-07a4-432e-bd0d-2a869c573161)

Nós nos concentraremos nos SNPs

## 1. Chamada com genoma de referência 
Com a redução dos custos de sequenciamento, cada vez é mais comum ver genomas de referência para muitas espécies. Está se tornando rotineiro em muitos projetos gerar o próprio genoma de referência e usá-lo para estudos populacionais posteriores. Por isso, começamos hoje aprendendo como fazer a chamada de variantes (SNPs) usando as sequências mapeadas contra o genoma de referência. 

### Programas para chamada de SNPs
Existem diversas ferramentas bioinformáticas desenvolvidas para identificar variantes genômicas a partir de dados de sequenciamento. Cada ferramenta tem suas abordagens, pontos fortes e requisitos, e a escolha ideal depende do tipo de projeto, da cobertura dos dados, da qualidade do genoma de referência e dos objetivos da análise.

Entre as ferramentas mais utilizadas, destacam-se:
1. **[GATK (Genome Analysis Toolkit)](https://gatk.broadinstitute.org/hc/en-us/articles/360037225632-HaplotypeCaller):** é considerado o padrão-ouro na genética humana e de populações. Ele utiliza um algoritmo sofisticado chamado HaplotypeCaller, que não olha apenas para uma base isolada, mas reconstrói localmente as sequências (haplótipos) para determinar com precisão se um SNP ou um pequeno InDel é biológico ou um artefato de alinhamento. No entanto, exige etapas complexas de pré-processamento (como realinhamento e recalibração de base) e depende de genomas de alta qualidade, o que pode ser um desafio em espécies não-modelo com genomas menos completos.
2. **[FreeBayes](https://github.com/freebayes/freebayes):** é uma alternativa similar a GATK, ja que é baseado em haplótipos. É frequentemente a ferramenta de escolha para genomas complexos devido à sua flexibilidade com ploidias variadas e sua capacidade nativa de analisar múltiplos indivíduos simultaneamente, utilizando o poder da probabilidade bayesiana para inferir a presença de polimorfismos na população como um todo. No entanto, ele também depende de genomas de alta qualidade. 
3. **[bcftools](https://samtools.github.io/bcftools/howtos/variant-calling.html):** é uma ferramenta rápido e de fácil uso, o bcftools é altamente recomendado para projetos com espécies não-modelo, genomas ainda em construção ou muito fragmentados. Ele oferece um fluxo simples, com poucos requisitos e excelente compatibilidade com outras ferramentas como samtools e vcftools. 

Como nosso foco são principalmente espécies vegetais comerciais, a maioria das quais já possui um genoma de referência de qualidade, aprenderemos a usar o GATK como ferramenta principal. 

### A. Pré-processamento antes de chamar SNPs
O GATK é extremamente exigente quanto aos metadados da referência. Antes de executá-lo, precisamos gerar um dicionário de sequências `(.dict)` e um índice FASTA `(.fai)`. 

```bash
# 1. Navegar até a pasta da referência
cd ~/workshop_bioinfo/data/reference

# 2. Criar o índice FASTA com Samtools
samtools faidx acrocomia_ref.fasta

# 3. Criar o Dicionário de Sequências com Picard
picard CreateSequenceDictionary \
    R=acrocomia_ref.fasta \
    O=acrocomia_ref.dict
```

💡*Opcional, mas recomendado*

Recalibra os arquivos bam em relação a um “conjunto verdadeiro” de SNPs

Os vcfs do conjunto verdadeiro são variantes nas quais temos alta confiança e tendem a ser gerados a partir de bibliotecas de alta cobertura sem PCR. Se tiver dispon[ivel, é recomendado fazer a recalibração. Na maioria das plantas não modelo, não temos esse tipo de dados, portanto, vamos pular essa etapa. 

### C. Chamada de variantes
A chamada de variantes não é um passo simples, pois o GATK faz a chamada de variantes por indivíduo. Posteriormente, é necessário consolidar em um único arquivo e chamar as variantes comuns entre todos os indivíduos. 

O primeiro passo é chamar as variantes para cada indivíduo. Nesta etapa, o GATK analisará cada arquivo .dedup.bam e gerará um arquivo .g.vcf.gz. O sufixo .g indica que o arquivo contém informações de confiança para todas as bases do genoma, não apenas para os polimorfismos. É um arquivo intermediário que precisamos usar antes de criar nosso arquivo VCF final.

```bash
# Criar diretório para os gVCFs
mkdir -p ~/workshop_bioinfo/results/gatk_gvcfs
cd ~/workshop_bioinfo/data/alignment

# Loop para processar todas as amostras de Acrocomia simultaneamente
for bam in *.dedup.bam; do
    SAMPLE=$(basename $bam .dedup.bam)
    echo "Gerando gVCF para a amostra: $SAMPLE"

    gatk HaplotypeCaller \
        -R ../../reference/acrocomia_ref.fasta \
        -I $bam \
        -O ../../../results/gatk_gvcfs/${SAMPLE}.g.vcf.gz \
        -native-pair-hmm-threads 4 \ 
        -ERC GVCF
done
Esta é a parte mais demorada de todo o processo. É hora de tomar um café ☕. 
```
💡 Verifique seu arquivo `gvcf` para garantir que ele tenha um arquivo de índice `.idx`. Se o `haplotypecaller` travar, ele produzirá um arquivo `gvcf` truncado que acabará travando a etapa `genotypegvcf`. 

O próximo passo é importar nossos arquivos `gvcf` para um arquivo `genomicsDB`. E uma representação compactada em banco de dados de todos os dados lidos em nossas amostras. O arquivo `GenomicsDB` contém todas as informações dos seus arquivos `GVCF`, mas não pode ser adicionado e não pode ser transformado de volta em um `gvcf`. Isso significa que, se você obtiver mais amostras, não poderá simplesmente adicioná-las ao seu arquivo `genomicdDB`, terá que voltar aos arquivos `gvcf`. 

💡 A ferrameta `GenomicsDBImport` substituiu o antigo `CombineGVCFs` porque utiliza um banco de dados estruturado, sendo infinitamente mais rápido e escalável para genomas grandes e populações numerosas. A grande particularidade computacional do `GenomicsDBImport` é que ele exige a definição de intervalos (cromossomos ou scaffolds) para particionar o trabalho, e utiliza um arquivo de mapeamento de amostras (Sample Map). Primeiro, devemos gerar estes dois arquivos

O algoritmo exige um arquivo tabular indicando o nome biológico da amostra e o caminho físico do seu respectivo gVCF (Sample Map).

```bash
# Navegar até o diretório dos gVCFs
cd ~/workshop_bioinfo/results/gatk_gvcfs

# Gerar o Sample Map automaticamente com um loop
for gvcf in *.g.vcf.gz; do
    SAMPLE=$(basename $gvcf .g.vcf.gz)
    # O comando 'echo -e' com '\t' insere uma tabulação (TAB) entre o nome e o arquivo
    echo -e "${SAMPLE}\t${gvcf}" >> sample_map.txt
done
```
A segunda particularidade é a necessidade estrita de paralelização espacial: ele precisa saber exatamente em quais regiões (intervalos) do genoma da *Acrocomia* ele deve atuar. Como nosso genoma está nos cromossomos, podemos indicar uma lista com o nome dos cromossomos. Isso a partir do nome do índice `(.fai)`. 

```bash
# Extrair a primeira coluna (nomes das sequências) do arquivo .fai para criar a lista
awk '{print $1}' ~/workshop_bioinfo/data/reference/acrocomia_ref.fasta.fai > intervalos_acrocomia.list

# Visualizar os primeiros intervalos gerados
head intervalos_acrocomia.list
```

Em seguida, chamamos `GenomicsDBImport` para criar o banco de dados

```bash
# Importar os dados para o GenomicsDB utilizando a lista de intervalos
gatk GenomicsDBImport \
    --sample-name-map sample_map.txt \
    --genomicsdb-workspace-path banco_dados_acrocomia \
    -L intervalos_acrocomia.list

# -----------------------------------------
# sample-name-map: é o nome de saída da biblioteca que estamos construíndo
#-L: Também pode ser usado para um único cromossomo. Substituímos a lista pelo nome da região de interesse. 
# -----------------------------------------
```

Com o `genomicsDB` criado, estamos finalmente prontos para identificar variantes e gerar um `vcf`.
```bash
# Realizar a chamada de variantes lendo diretamente do banco de dados recém-criado
gatk GenotypeGVCFs \
    -R ~/workshop_bioinfo/data/reference/acrocomia_ref.fasta \
    -V gendb://banco_dados_acrocomia \
    -O ../acrocomia_populacao_bruta.vcf.gz
```
❓Vamos fazer um less neste arquivo. Como ele fica? Que variantes você consegue ver? 


O arquivo gerado ao final deste processo `(acrocomia_populacao_bruta.vcf.gz)` é categorizado como "bruto" (unfiltered). Ele contém todos os candidatos a polimorfismos. A próxima e mandatória etapa analítica é a filtragem criteriosa (Hard Filtering).

Como estamos interessados apenas em SNPs, o primeiro passo é selecionar apenas essas marcas. 
```bash
gatk SelectVariants \
-R reference.fasta \
-V acrocomia_populacao_bruta.vcf.gz \
--select-type SNP \
-O raw_snps.vcf.gz
```
 
### C. Filtragem de SNPs
Como podem ver, o arquivo final é chamado `raw_snps.vcf.gz`. Isso porque é um arquivo final sem filtros de qualidade nem nada. O GATK tem a função `VariantFiltration` que permite fazer esses filtros. 

No entanto, existem outras duas ferramentas mais comuns e utilizadas para realizar esta etapa: `bcftools` e `VCFtools`. Iremos utilizá-las na próxima prática para realizar os filtros no nosso VCF e deixar o arquivo final pronto para as nossas análises. 


## 2. Chamada com genoma de referência (Alternativa)


## 3. Chamada sem genoma de referência (_de novo_)
