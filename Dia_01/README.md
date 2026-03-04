# Guia Prático - Dia 1: Setup e Exploração de Dados
Este roteiro foca na organização inicial do projeto e na manipulação de arquivos de sequenciamento via linha de comando.

## [1. Comandos básicos em Linux](Dia_01/comandos_basicos.md) 
## [2. Organização do Workspace](Dia_01/organizacao)

## 1. Organização do Workspace
O primeiro passo é criar uma estrutura de diretórios que garanta a reprodutibilidade.

```bash
# 1. Criar a pasta principal do projeto
mkdir -p ~/workshop_bioinfo && cd ~/workshop_bioinfo

# 2. Criar a estrutura de pastas para todos os dias
mkdir -p data/raw data/processed/trimmed data/processed/aligned
mkdir -p results/qc results/vcf_raw results/vcf_filtered
mkdir -p scripts logs docs

# 3. Criar arquivos README de navegação rápida
echo "Dados brutos (imutáveis)" > data/raw/README.txt
echo "Scripts e códigos utilizados" > scripts/README.txt
echo "Resultados finais e figuras" > results/README.txt

# 4. Verificar a estrutura
ls -R
