# Lab3: Finding homologs with BLAST

Copyright Joshua Rest. Do not post this lab anywhere online.

# 1. We will search for homologs in proteomes from several species. 

## The Species

The goal of today's lab is to use BLAST to find homologous sequences (orthologs and paralogs) to GQ-Coupled Rhodopsin in bilaterians. Then, you will repeat the same task for your gene family.

Today, we will work with nine animals: 5 protostomes and 4 deuterostomes. Read a little about each:  
*Homo sapiens  
Branchiostoma belcheri  
Mizuhopecten yessoensis  
Acanthaster planci  
Ciona intestinalis  
Drosophila melanogaster  
Adineta vaga  
Echinococcus granulosus  
Lingula anatina*  

>[Brightspace] 1. Match each latin name with its common name.  (Use google to find your answers.)

>[Brightspace] 2. What phylum are each of these species in?  (Use google to find your answers.)

>[Brightspace] 3. Which species are protostomes and which species are deuterostomes?  (Use google to find your answers.)

## Working with Proteomes

Instead of working with genomes, we are going to work with proteomes. 
>[Brightspace] 4. What is a proteome? 

Not only will working with proteomes make our life easier, but we are looking at some ancient divergences (>500 million years), so nucleotide sequences are likely saturated and not very helpful here. Protein sequences, on the other hand, are functionally constrained and therefore tend to evolve at a slower rate, making distant comparison more reasonable. 

We are going to use proteomes files that have only *one isoform per gene* selected. **Rest has provided single-isoform proteomes already included in this lab3 GitHub repository.** For human, lots of information, such as gene expression, was taken into account when choosing the isoform; this is called "RefSeq select." For the other proteomes, Rest filtered out the longest isoform. He also reformatted the definition lines to: "species.geneid_annotation". 

We are going to create a database for BLAST to use from these sequences. BLAST will  align your query sequence with each of the sequences in the database, and then return high scoring pairs (HSPs).

# 2. Get set up.
## Make a copy of this README.md over at HackMD
That is an easy place to edit this file.

## Fire up your EC2 instance and log in.
![enter image description here](https://res.cloudinary.com/apideck/image/upload/v1531054194/catalog/amazon-ec2/icon128x128.png)![enter image description here](https://miro.medium.com/max/256/0*en_6yjFaxJ0H0kBK.png) 

Start a screen session; this can be useful:
```
screen -dRR
```

## Clone Lab 3

**You only need to do this ONE time.**

On the command line, clone the lab 3 repository. Look at the URL, above, of this GitHub repository. It will look something like: https://github.com/Bio312/lab3-myusername
NOTE again that myusername below is your github username. 

    cd ~/labs
    git clone https://github.com/Bio312/lab3-myusername

You will be asked to enter your GitHub username and password. Use the token you generated instead of your password. Git will now clone a copy of today's lab into a folder called lab3-myusername (where myusername is your GitHub username). Go there:

    cd lab3-myusername

Take a look at what is in the folder using the `ls` command. At the end of the lab, you will push the changes and files back into the online repository.

**Do all the work for this lab in this new folder you have cloned.** Check this at any time by typing ```pwd```.

## Thinking about errors.

It is common to make errors when working at the terminal. Don't panic! By carefully working your way backwards and looking at where you are, and what's there, you should be able to figure it out.

**THE MOST COMMON ERROR THAT YOU WILL ENCOUNTER IS WHEN YOU ENTER A COMMAND BUT YOU ARE IN THE WRONG DIRECTORY**  
**HERE'S WHAT YOU SHOULD DO WHEN ENCOUNTERING AN ERROR:**
> 1. Carefully read the error message. What is it telling you? You can google the error message if you don't understand.
> 2. Look back the previous few steps in the lab. What files are you looking for? Where do you expect those files to be?
> 3. Find out what directory you are currently in by typing `pwd`. Is this where you expect to be? If not, move there.
> 4. Find out what files are in the directory by typing `ls`. Is the file you are expecting there? You can also look for a specific filename by typing `ls filename`. If the file is missing, think about why, and work your way backwards in the lab to figure it out.

>[Brightspace] 5. What should you do when encountering an error?

## Create the BLAST database

**You only need to do this ONE time.**

Within the cloned folder is another folder `proteomes`, which contains the compressed proteomes that will be turned into a BLAST database today. We are then going to concatenate these proteomes into a single file (If you don't know what the word "concatenate" means, please look it up.)

First, go to your lab 3 folder:
```bash
cd ~/labs/lab3-myusername
```
Next, uncompress the proteomes. Run the following command:
```bash
gunzip proteomes/*.gz
```
Now, put all the protein sequences into a single file using the cat command:
```bash
cat  proteomes/* > allprotein.fas
```
Now we want to perform a BLAST search to find potential homologs of a query protein. Before you do this, you need to build a blast database with the proteomes we just concatenated. Read about it this [here](https://www.ncbi.nlm.nih.gov/books/NBK279688/).

Type in the command to make the BLAST database:
```bash
makeblastdb -in allprotein.fas -dbtype prot
```

# 3. BLAST a GQ-coupled rhodopsin protein against the database to identify homologs.

Now that our database is set up, we will work on our blast search.

## Create a new working directory
You will be conducting TWO blast searches today: one on GQ-coupled rhodopsin and one for your protein. In order to keep things separate, create a folder for the GQ-Rhodopsin BLAST search using the `mkdir` command:

```bash
mkdir /home/ec2-user/labs/lab3-myusername/gqr
```

>[Brightspace] 6. What does the `mkdir` command do?

Now go to this folder:

```bash
cd /home/ec2-user/labs/lab3-myusername/gqr
```

>[Brightspace] 7. What does the `cd` command do?

Check where you are with the command `pwd`.
```bash
pwd
```

>[Brightspace] 8. What did the command `pwd` return?

**Future shortcut:** The location`/home/ec2-user` can also be represented by a tilda ``~``. So, the following command is the same: `cd ~/labs/lab3-myusername/gqr`

Check to make sure you are in the correct folder. If you type `pwd`, you should see: 
```/home/ec2-user/labs/lab3-myusername/gqr```

## Download your query protein
We will use a GQ-coupled rhodopsin protein from *Mizuhopecten yessoensis* as our query sequence. that we used in labs 1-2. You can find this in your lab2 (LOC5514051.X1.aa.fa) folder. Or, you can download it directly using the following command:
```bash
ncbi-acc-download -F fasta -m protein XP_021373098.1 
```

>[Brightspace] 9. How can you check to see if the resulting file exists, and if contains the expected information?

>[Brightspace] 10. If you want to download a protein with a different accession, what part of the command would you change?

## Perform the BLAST search 
Now, perform a blast search using the query protein:
```bash
blastp -db ../allprotein.fas -query XP_021373098.1.fa -outfmt 0 -max_hsps 1 -out gqr.blastp.typical.out
```
>[Brightspace] 11. Which part(s) of this command should you change if you wanted to use a different query protein?

## Exploring the BLAST results
Look at the output in ``gqr.blastp.typical.out`` using ``less``.
Scroll down to see some of the alignments of high scoring pairs. This is similar to the output shown on BLAST's web page.

>[Brightspace] 12. What is the top scoring hit in the BLAST results?

>[Brightspace] 13. What is the annotation of the top hit to this query in _Homo sapiens_? 

>[Brightspace] 14. What range of expect-values (E-values) are included in this output, and how should they be interpreted?

## Perform a BLAST search, and request tabular output
Let's create a more detailed and easier-to-process output of the same analysis. The -outfmt flag specifies a particular output format that will be useful for our analysis. Read about output formats [here](http://www.metagenomics.wiki/tools/blast/blastn-output-format-6).

Type in the following command:
```bash
blastp -db ../allprotein.fas -query XP_021373098.1.fa -outfmt "6 sseqid pident length mismatch gapopen evalue bitscore pident stitle"  -max_hsps 1 -out gqr.blastp.detail.out
```
>[Brightspace] 15. Which part(s) of this command should you change if you wanted to use a different query protein?

 Take a look at the output in ``gqr.blastp.detail.out`` using the ``less -S`` command.

>[Brightspace] 16. How many total human hits are in the file?  
You can use this command to avoid counting by hand: 
```bash
grep -c Hsapiens gqr.blastp.detail.out
```
>[Brightspace] 17. What part of the command would you change if you wanted to count the hits for another species?  
>[Brightspace] 18. What part of the command would you change if you wanted to count the Human hits in the output from a different blast search?  
  
## Filtering the BLAST output for high-scoring putative homologs

Next we need to choose which putative homologs to include. We only want to include high-scoring matches for our analysis.

[This article describes how to choose e-value cutoffs.](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3820096/)

Let's require the e-value to be less than 1e-35 . 

>[Brightspace] 19. Why might we want to only keep e-values that are less than 1e-35?

Use this command you to filter our output file to satisfy this requirement:

```bash
awk '{if ($6< 1e-35 )print $1 }' gqr.blastp.detail.out > gqr.blastp.detail.filtered.out
```
>[Brightspace] 20. What is the awk program? (use google to find out)  
>[Brightspace] 21. What part of the command would you change if you wanted to include more hits, by filtering out all hits less than 1e-30?  
>[Brightspace] 22. What part(s) of the command should you change if you want to filter a different blast output file?  

Here's an easy way to count the total number of hits in the BLAST results after the filter. We will use the wc command (read about it at: https://shapeshed.com/unix-wc/
```bash
wc -l gqr.blastp.detail.filtered.out
```
>[Brightspace] 23. What does the -l flag do to the wc command? (look using `man wc`)  
>[Brightspace] 24. How many total hits were removed by this filter?  

**Now, we have used BLAST to identify a set of putative homologs to the query that we want to study.**

NOTE: For your project, it is desirable to work with between 25 and 60 homologs. Rest has pre-selected gene families that should be in that range. When you repeat this for your own gene, if you are not in that range, you will need to change the e-value threshold to increase or decrease the number of hits  - but you **must** talk to your TA when making this decision.  

### How many paralogs are in each species?
It is useful to know how may paralogs are found in each species.
```bash
grep -o -E "^[A-Z][a-z]+\." gqr.blastp.detail.filtered.out  | sort | uniq -c
```
>[Brightspace] 25. Indicate the number of paralogs found in each species.  
>[Brightspace] 26. What part of this command would you change to repeat this for a different blast output file?  

# 4. Investigate and read about your assigned gene family.
You have been assigned a gene family to study this semester. 
Rather than tell you the name of the gene family, I have provided you with one member of the gene family (StartingGene) as a query to find the rest of the gene family members, which have similar sequences. You can find your assigned StartingGene under the StartingGene entry in your Grades section of Brightspace.
```diff
-[GitHub] What is the name of your starting gene.
```

Now, let's start to figure out what gene family your StartingGene belongs to. **Look back to Lab1, where we figured out the gene family that GQ-coupled rhodopsin belongs to.** You will perform similar steps here.

There are many ways to try and figure this out, and there may be many overlapping defintions of a gene family. We will continue to learn about these throughout the semester. Today we just want to get some idea of a place to start  thinking about your gene family.

Hint: This gene family has homologs (gene copies) in many bilaterian animal species, including humans. But a gene family means that there are also multiple related genes within a species' genome.

Here's a good place to start:

>1. Go to Genbank [gene](https://www.ncbi.nlm.nih.gov/gene/) page, and search for the accession of your StartingGene. 
Find the description of the gene on the gene page. What is it?
```diff
-[GitHub] What is the **description** of the gene?
-[GitHub] Is there any other information present that gives you a guess at the name of the gene family?
```

>2. Go to scholar.google.com; type in the **description** from NCBI and the phrase "gene family" (in quotes). Look over the first several articles. Look for information that includes terms in your StartingGene's description, and about the name of the gene family is belongs to. You might want to try adding or subtracting search terms. Other terms you could try: paralog, evolution, bilateria.

```diff
-[GitHub] List at least one research article that you used to investigate the identity of your assigned gene family:
-[GitHub] What gene family do you think your StartingGene belongs to? Describe what you know about related subfamilies, families, or superfamilies.
```

# 5. Find and align YOUR gene family homologs by using your "StartingGene" as the query sequence.

Starting over again **repeat Part 3**, using the same steps to find and align gene family homologs using **your "StartingGene" as the query sequence**. (Note: DO NOT re-clone the lab (part 1). DO NOT re-create the database. Just redo Part 3, but using your starting gene as the query.)

**Think carefully, for each command, about what you need to change about the command to make it work for *your* gene family.** For example, we just used the term *gqr* in some of the **file names**. This refers to the GQ-coupled rhodopsin(for example) that is the description of our protein family. **If you use the exact same file names, then your file names won't make any sense!** Work * carefully* to change filenames to reflect the genes or gene family **you** are working on!

Note that, at the beginning of part 3, we created a folder called "gqr" to work in. **Please create a folder with a different name to work in so that all of the files for your gene family analysis are there.**
```diff
-[GitHub] What did you call this new directory?

-[GitHub] Change into the directory, and type the command 'pwd'. Paste the output here. 
```

It is REALLY important to keep track of the  commands you are using for your own gene family here! This is why we are using hackmd - it's a nice place to write this all down. **Having your new commands in this document will be checked as part of your repository check, worth 10 points!**

You don't have to write brand new commands. But copy the commands that we used for GQ-coupled rhodopsin above and **change them so they work for your own gene family!**

```diff
-[GitHub] Describe how you needed to change the commands and filenames to work for  your own gene family, without overwriting your previous results.

-[GitHub] Keep track of all commands you use for your own gene family! Paste them along withy any notes below! These commannds are REQUIRED to get full credit for your repository.
```
 
 When you have finished, answer the following questions:
```diff
-[GitHub] How many proteins do you have in your BLAST results after filtering for an e-value of 1e-35?

-[GitHub] How many homologs does each of the 9 species have?

-[GitHub] Do any of the species have zero homologs?
```
**IMPORTANT NOTE:** For your project, it is desirable to work with between 25 and 65 homologs. In addition, you should have between 2 and 15 copies from each species. Rest has pre-selected gene families that should be in these ranges. If you are not in that range, you will need to change the e-value threshold to increase or decrease the number of hits  - but you **must** talk to your TA when making this decision.  

>[Brightspace] 27. Show your TA the table of the # of homologs in each species. They will give you a code-word once they have looked at it.
 
# 6. Wrap up

## Save your history:

      history > lab3.commandhistory.txt

  
##  Push any new files to the remote Git repository.

Add all of your results to the repository, commit them, and push them to the remote repository at GitHub.com

First, ensure you are in the main lab3 directory:
```
cd ~/labs/lab3-myusername`
```
Now, commit your changes. **Make sure each line works before you continue with the next line.**

```bash
find . -size +5M | sed 's|^\./||g' | cat >> .gitignore; awk '!NF || !seen[$0]++' .gitignore
```
```bash
git add .
```
```bash
git commit -a -m "Adding all new data files I generated in AWS to the repository."
```
```bash
git pull --no-edit
```
Did all of the above work? **Make sure!** If so, then run the following command:
```bash
git push 
```

