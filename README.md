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
platanus assemble -f sub1.fastq.trimmed  sub2.fastq.trimmed -o aaa
```
9. Сбор скаффолдов:
```
platanus scaffold -c aaa_contig.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matep1.fastq.int_trimmed matep2.fastq.int_trimmed -o aaa
```
10. Удаление исходных файлов:
```
rm sub* matep*
```
11. Удаление подрезанных чтений:
```
rm sub*.trm matep*.int_trm
```
## Colab
1. импорт библиотек
```
import re
```
2. функция для получения данных
```
def get_info(f, text, output = True):
    lengths = []
    total_len = 0
    num = 0
    max_len = 0
    length = 0
    score = 0
    max_sequence = ''
    curr_sequence = ''
    for line in f:
        if (line[0] == '>'):
            if num != 0:
                lengths.append(length)
            num += 1
            if length >= max_len:
                max_len = length
                max_sequence = curr_sequence
            curr_sequence = ''
            length = 0
        else:
            curr_sequence += line.strip()
            length += len(line.strip())
            total_len += len(line.strip())
     
    lengths.sort(reverse = True) 
    for i in lengths:
        score += i
        if score >= total_len / 2:
            if output == True:
                print(f'Анализ {text}\n\
Общее количество: {num},\n\
Общая длина: {total_len},\n\
Длина самого длинного: {max_len},\n\
N50: {i}\n')
            break
    return max_sequence
```
3. информация о контигах:
```
max_cont = get_info(open('aaa_contig.fa', 'r'), 'Контигов')
```
Анализ Контигов

Общее количество: 614,

Общая длина: 3925500,

Длина самого длинного: 179307,

N50: 55038

4. ..о скаффолдах:
```
max_scaf = get_info(open('aaa_scaffold.fa', 'r'), 'Скаффолдов')
```
Анализ Скаффолдов

Общее количество: 66,

Общая длина: 3876492,

Длина самого длинного: 3838403,

N50: 3838403

5. подсчет гэпов: 
```
print(f'Общая длина гэпов: {max_scaf.count("N")}')
max_scaf = re.sub(r'N{2,}', 'N', max_scaf)
print(f'Число гэпов: {max_scaf.count("N")}')
```
Общая длина гэпов: 7048

Число гэпов: 58

6. сокращение гэпов:
```
platanus gap_close -c aaa_scaffold.fa -IP1 sub1.fastq.trimmed sub2.fastq.trimmed -OP2 matep1.fastq.int_trimmed matep2.fastq.int_trimmed -o aaa
```
7. открываем файл с подрезанынми чтениями:
```
max_scaf = get_info(open('aaa_gapClosed.fa', 'r'), 'Скаффолдов', False)
```
8. подсчет гэпов для подрезанных чтений:
```
print(f'Общая длина гэпов для обрезанных чтений: {max_scaf.count("N")}')
max_scaf = re.sub(r'N{2,}', 'N', max_scaf)
print(f'Число гэпов для обрезанных чтений: {max_scaf.count("N")}')
```
Общая длина гэпов для обрезанных чтений: 1471

Число гэпов для обрезанных чтений: 7

