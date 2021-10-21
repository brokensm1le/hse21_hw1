# hse21_hw1

1) Создаем папку с домашним заданием и линкуем нужные сырые данные:
```
mkdir hw
cd hw
mkdir 1
cd 1
ls -1 /usr/share/data-minor-bioinf/assembly/* | xargs -tI{} ln -s {}
```
2) Делаем случайные чтения:
```
seqtk sample -s1025 oil_R1.fastq 5000000 > pe_R1.fastq
seqtk sample -s1025 oil_R2.fastq 5000000 > pe_R2.fastq
seqtk sample -s1025 oilMP_S4_L001_R1_001.fastq 1500000 > mp_R1.fastq
seqtk sample -s1025 oilMP_S4_L001_R2_001.fastq 1500000 > mp_R2.fastq
```
3) Оцениваем качество для наших чтений(fastQC и multiQC):
```
mkdir fastqc
ls *.fastq | xargs -P 4 -tI{} fastqc -o fastqc {}

mkdir multiqc
multiqc -o multiqc fastqc
```
Для скачивания файлов с сервера использовал программу FileZilla

4) Обрезаем чтение по качеству и убираем праймеры:
```
platanus_trim pe_R1.fastq pe_R2.fastq 
platanus_internal_trim mp_R1.fastq mp_R2.fastq  
```
Удаляем не нужные файлы
```
ls -1 *.fastq | xargs -tI{} rm -r {}
```
5) Оцениваем качество для наших новых(обрезанных) чтений(fastQC и multiQC):
```
mkdir trimmed_fastq
mv -v *trimmed trimmed_fastq/
```
```
mkdir trimmed_fastqc
ls trimmed_fastq/* | xargs -P 4 -tI{} fastqc -o trimmed_fastqc {}
```
```
mkdir trimmed_multiqc
multiqc -o trimmed_multiqc trimmed_fastqc
```
До:
![alt](./pictures/stat_1.png)
После:
![alt](./pictures/stat_2.png)



6) Cобираем контиги из подрезанных чтений:
```
time platanus assemble -o Poil -f trimmed_fastq/pe_R1.fastq.trimmed trimmed_fastq/pe_R2.fastq.trimmed 2> assemble.log
```
7) Анализ полученных контигов:

jupyter_notebook

8) Собираем скаффолды:
```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 trimmed_fastq/pe_R1.fastq.trimmed  trimmed_fastq/pe_R2.fastq.trimmed -OP2 trimmed_fastq/mp_R1.fastq.int_trimmed trimmed_fastq/mp_R2.fastq.int_trimmed 2> scaffold.log
```
9) Анализ полученных скаффолдов + количество гэпов:

jupyter_notebook

Создаем файл с одним самым большим размером:
```
echo scaffold1_len3834575_cov231 > name_scaff.txt
seqtk subseq Poil_scaffold.fa name_scaff.txt > BigScaff.fna
rm -r name_scaff.txt
```

10) Уменьшаем кол-во гэпов:
```
time platanus gap_close -o Poil -c Poil_scaffold.fa -IP1 trimmed_fastq/pe_R1.fastq.trimmed  trimmed_fastq/pe_R2.fastq.trimmed -OP2 trimmed_fastq/mp_R1.fastq.int_trimmed trimmed_fastq/mp_R2.fastq.int_trimmed 2> gapclose.log
```
11) Анализ полученных скаффолдов с уменьшением гэпов:

jupyter_notebook
```
echo scaffold1_cov231 > name_scaff.txt
seqtk subseq Poil_gapClosed.fa name_scaff.txt > longest.fna
rm -r name_scaff.txt
```
