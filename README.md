# hse22_hw1
1. Создание ссылок на файлы:
```
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```
2. Выбор случайных чтений (Seed 326)
```
seqtk sample -s326 oil_R1.fastq 5000000 > sub1.fastq
seqtk sample -s326 oil_R2.fastq 5000000 > sub2.fastq
seqtk sample -s326 oilMP_S4_L001_R1_001.fastq 1500000 > matep1.fastq
seqtk sample -s326 oilMP_S4_L001_R2_001.fastq 1500000 > matep2.fastq
```
![image](https://user-images.githubusercontent.com/114301236/194422400-e575f9d7-bec2-4bb8-8b41-c511a0c7c0d9.png)

3. Оценка чтений FastQC
```
mkdir fastqc
ls sub* matep* | xargs -tI{} fastqc -o fastqc {}
```
![image](https://user-images.githubusercontent.com/114301236/194422626-ea75f2dc-bbb6-43e7-b1f7-c7e9fe6cc489.png)

4. Отчёт MultiQC
```
mkdir multiqc
multiqc -o multiqc fastqc
```
![image](https://user-images.githubusercontent.com/114301236/194425450-d2554fdc-5f74-44f4-a787-5404a275cfc9.png)
![image](https://user-images.githubusercontent.com/114301236/194425882-2ec44050-88cc-4b07-88ae-9a2bb82bda0a.png)
![image](https://user-images.githubusercontent.com/114301236/194425943-8811f4d4-b8a3-4d1d-93a0-3ad16dd15a14.png)

5. Обрезка чтений:
```
platanus_trim sub*
platanus_internal_trim matep*
```
![image](https://user-images.githubusercontent.com/114301236/194426614-0b1f7e3b-84dd-4754-b201-22ca47bb22f0.png)

6. Оценка обрезанный чтений с помощью FastQC
```
mkdir fastqc_trm
ls sub* matep*| xargs -tI{} fastqc -o fastqc_trm {}
```
7. Отчет для обрезанных чтений MultiQC
```
mkdir multiqc_trm
multiqc -o multiqc_trm fastqc_trm
```
![image](https://user-images.githubusercontent.com/114301236/194429228-f7a8d69b-5725-4218-914f-92c8770ff43d.png)
![image](https://user-images.githubusercontent.com/114301236/194429391-8347e79b-2b6c-49b4-a42c-2910c25160cc.png)
![image](https://user-images.githubusercontent.com/114301236/194429418-b8f34801-7d8e-47d3-8074-192c5fb727b0.png)

8. Сбор контигов:
```
time platanus assemble -o Poil -f sub1.fastq.trm sub2.fastq.trm 2> assemble.log
```
9. Сбор скаффолдов:
```
time platanus scaffold -o Poil -c Poil_contig.fa -IP1 sub1.fastq.trm sub2.fastq.trm -OP2 matep1.fastq.int_trm matep2.fastq.int_trm 2> scaffold.log
```
10. Удаление исходных файлов:
```
rm sub* matep*
```
11. Удаление подрезанных чтений:
```
rm sub*.trm matep*.int_trm
```

