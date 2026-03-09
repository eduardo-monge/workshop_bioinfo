# 💻 Prática Dia 3 (tarde): Filtros do arquivo VCF

A última etapa antes de iniciar as análises populacionais é filtrar o VCF final. Isso é feito para garantir que os SNPs sejam de alta qualidade e padronizados entre todas as amostras. 

---
## 0. Lembrando do arquivo VCF
O `Variant Call Format (VCF)` é o o formato universal que armazena toda a variabilidade genética e informação de variantes estructurais. Estruturalmente, um arquivo VCF divide-se em dois grandes blocos hierárquicos:
1. **Cabeçalho (Header)**: Um extenso bloco inicial de metadados, onde cada linha obrigatoriamente começa com ##. Ele atua como um dicionário, definindo de forma rigorosa as abreviações estatísticas, os filtros aplicados, os formatos dos dados e a versão das ferramentas bioinformáticas utilizadas na geração do arquivo.
2. **Matriz de Variantes (Data Table)**: Inicia-se com a linha de cabeçalho das colunas (marcada por um único #CHROM). A partir deste ponto, cada linha subsequente descreve detalhadamente um único polimorfismo (SNP ou InDel) encontrado na população.

O cabeçalho do VCF segue esse padrão:
| Campo | Descrição Acadêmica |
| :--- | :--- |
| **`#CHROM`** | O cromossomo ou *scaffold* de referência onde o polimorfismo está ancorado. |
| **`POS`** | A coordenada genômica exata (posição física em pares de bases) da mutação. |
| **`ID`** | Identificador alfanumérico da variante (ex: em humanos, o número dbSNP). Em espécies não-modelo sequenciadas *de novo*, este campo geralmente recebe um ponto (`.`). |
| **`REF`** | O Alelo de Referência (a base nucleotídica ancestral ou presente no genoma de referência). |
| **`ALT`** | O Alelo Alternativo (a base mutada inferida na nossa população amostrada). |
| **`QUAL`** | Escore de qualidade (*Phred-scaled*) da chamada da variante. Valores maiores indicam uma maior probabilidade estatística de que o polimorfismo seja biologicamente real e não um erro técnico. |
| **`FILTER`** | Status de aprovação na filtragem de qualidade (*Hard Filtering*). Variantes validadas recebem a etiqueta **`PASS`**. Variantes reprovadas exibem o nome do filtro que falhou (ex: `LowQual`, `StrandBias`). |
| **`INFO`** | Metadados e métricas globais consolidadas do *locus* (ex: profundidade total da coorte, frequência alélica da população). |
| **`FORMAT`** | Um dicionário (ex: `GT:AD:DP:GQ:PL`) que define a estrutura exata dos dados genotípicos contidos nas colunas individuais à direita. |
| **`<amostras>`** | Colunas terminais nomeadas com o ID de cada indivíduo (ex: `Acro_01`, `Acro_02`). Contêm o genótipo inferido (`0/0` homozigoto referência, `0/1` heterozigoto, `1/1` homozigoto alternativo) e suas métricas locais, seguindo a regra ditada na coluna `FORMAT`. |
<img width="1504" height="619" alt="vcf_format" src="https://github.com/user-attachments/assets/cbda7aa1-e532-4f98-a0ec-a1029d461076" />


Para manipular os arquivos VCF existem duas ferramentas muito utilizadas: 
1. **[BCFtools](https://www.htslib.org/doc/1.0/bcftools.html)**:  é a ferramenta de eleição para a manipulação técnica e estrutural da matriz de dados. Ele permete conversão de formatos (de VCF textual para BCF binário comprimido), concatenação e fusão de múltiplos cromossomos, indexação rápida e a aplicação de filtros técnicos complexos baseados nas anotações matemáticas da coluna `INFO` (como o *Mapping Quality* ou *Strand Bias*).
2. **[VCFtools](https://vcftools.github.io/man_latest.html)**: Ele é mais antigo, e tem alguns friltros não disponíveis no BCFtools, muitos com sentido biológico importante. Utilizamos o VCFtools para expurgar genótipos com excesso de dados faltantes (*Missing Data*) e remover alelos excessivamente raros (*Minor Allele Frequency* - MAF).


## 1. Filtrando com bcftools
O nosso objetivo é filtrar os VCF gerados com GATK e FreeBayes para reter apenas SNPs:
+ bialélicos
+ MAF ≥ 0.05
+ presentes em ≥60% das amostras
+ remover SNPs com >50% de missing data
+ calcular estatísticas finais do VCF


Utilizaremos o `bcftools view` para reter estritamente os SNPs e forçar a exclusão de qualquer *locus* que possua mais de um alelo alternativo.

```bash
bcftools view \
-v snps \
-m2 -M2 \
-q 0.05:minor \
-i 'F_MISSING<=0.4' \
raw_variants.vcf.gz \
-Oz \
-o filtered_snps.vcf.gz


```









