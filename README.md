# gffInsert



Based on the novel_class_codes = {'r', 'u', 'i', 'y', 'p'} specified in the XXX.combined/annotated gtf generated by the gffcompare process (using query gtf from stringtie and reference gtf as inputs), I have written a Python script to append these novel transcripts to the reference gtf. Please see the attached append.py file for details.

Secondly, since inserting potential overlap transcripts (class_codes = {'k', 'm', 'n', 'j', 'e'}) into the appropriate location in the reference gtf involves many search processes, I wrote a function called parse_gtf_to_dict (see attached file) to reduce the O(n) time complexity of future searches to O(1). For example, in the reference gtf, the unique gene ID will be the key in this JSON file, and the gene, transcript, exon, and CDS will be the values.

![image](https://github.com/dxu104/gffInsert/assets/90865804/485cd245-11d0-4045-8324-e6f269fc3c64)

We also have a function to convert JSON back to gtf format (see attached parse_json_gtf.py).

Next, I wrote a script called find_overlap_transcript.py to gather all transcripts and exons with class_codes = {'k', 'm', 'n', 'j', 'e'} from the XXX.combined.gtf. Each transcript and its subsequent exons are considered as a whole and are inserted into the reference gtf file. The insertion location is based on the cmp_ref "AMEX60DDU001003602.3" attribute in the 9th field of the gtf file. This cmp_ref serves as an excellent example to demonstrate the insertion and sorting process. 

In the below reference gtf screenshot, multiple transcripts under "AMEX60DDU001003602" are placed in increasing order by their start position (4th field of gtf). Each transcript and its subsequent exons are inserted into the reference gtf based on the same gene ID and start position. Sometimes, in the combined.gtf, two or more potential overlap transcripts and exons may share a similar cmp_ref, like "AMEX60DDU001003602.3", "AMEX60DDU001003602.4", "AMEX60DDU001003602.7". All these transcripts and exons are inserted and sorted simultaneously.

![image](https://github.com/dxu104/gffInsert/assets/90865804/849375f4-5951-409c-af99-a352c3849506)

To implement this, I wrote a function called parse_gtf_to_dict_by_cmp_ref.py. In the output file, the unique cmp_ref attribute is the key, and the transcripts and exons are the values. All values in this output JSON file will be correctly inserted into the reference gtf based on the same gene ID (key) and start position.

![image](https://github.com/dxu104/gffInsert/assets/90865804/9ef145e0-cd49-460d-af12-d755655ac9e4)

![image](https://github.com/dxu104/gffInsert/assets/90865804/492fc186-a066-4340-9292-60e2824a00d9)

The screenshot above shows that even under the same cmp_ref, different STRG numbers can result in multiple transcripts. A similar situation occurs in the reference gtf (multiple transcripts under the same gene ID). Thus, I wrote separate_transcripts_with_same_cmp_ref_geneID, where for transcripts and exons, the start_position is the key. For gene or CDS entries, since their start positions are irrelevant to the insertion position, we just set random but unique keys for each.

![image](https://github.com/dxu104/gffInsert/assets/90865804/91f098a1-8e64-42b4-b959-98dedf011e54)

Since the transcripts below share the same start_position as the screenshot below, we combine them into one key-value pair. Then we can insert these two transcripts together into the reference gtf.

![image](https://github.com/dxu104/gffInsert/assets/90865804/68dc7526-fd0b-4866-8c82-f6267b2b8dbe)


Finally, we start the insertion sort part (see attached insert_by_start_position.py). The specific insertion sort algorithm involves inserting an ordered array A into another ordered array B. First, a number A[i] is inserted into the ordered array B. If A[i] is found to be smaller than a value B[j] , it is inserted before B[j]. If no such position is found, it implies that the number A[i] is the largest, so A[i] should be appended at the end of the last transcript and exon combination in the already traversed part of B.



Each insertion is based on the current state of the data, which is an important characteristic of the insertion sort algorithm. The elements of array A are the transcript start positions from a specific gene in the XXX.annotation/combined.gtf file, while the elements of array B are the transcript start positions from the same gene in the reference .gtf file.

The last step is to use the parse_json_to_gtf script to generate the final annotation file. 

Additionally, the code offers excellent reusability, allowing modifications to class_codes as needed. Running all the above steps on one 16 GB 2400 MHz DDR4 and 2.6 GHz 6-Core Intel Core i7 takes 40 second, demonstrating efficiency and effectiveness.
