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

### Programas para chamada de SNPs



## 2. Chamada sem genoma de referência (_de novo_)
