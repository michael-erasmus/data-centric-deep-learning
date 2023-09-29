# Task 1 - Annotation Data

## Step 1 - Generating example sentences

I started with this prompt in ChatGPT

PROMPT:
```
Generate 90 sentences in a single list similar to the following, just the sentences nothing else

The athlete won a gold medal at the Olympics.
The legislation was passed after a long debate in the Senate.
The discovery of the Higgs boson at CERN marked a milestone in particle physics.
The swimmer broke the world record in the 100-meter freestyle.
The bill received bipartisan support and was signed into law by the president.
The recent findings on black holes have revolutionized our understanding of space-time.
The spacecraft successfully docked with the International Space Station.
The gymnast captured the audience's hearts and took home the bronze medal.
After months of negotiations, the climate accord was finally agreed upon by the majority of nations.
Scientists successfully cloned a sheep, sparking ethical debates worldwide.
```

RESULTS (Excerpt):
```
Here are 90 sentences in the desired style:

The marathon runner achieved a personal best in the city marathon.
The amendment to the constitution was approved after rigorous discussions in the assembly.
The detection of gravitational waves opened a new chapter in astrophysics.
```

I saved the output to a file called `input_sentences.csv`

## Step 2 - Format data for annotation prompt

I then used ChatGPT's 'Advanced Data Analysis' mode to help write a script to format the data into the annotation prompt.

Here's the prompt I used (along with the uploaded `input_sentences.csv` file)

PROMPT:
```markdown
Here's a csv file with list of sentences. Run a script to create a new output text file where each input sentence is transformed like this

Task:
Input: <input_sentence>
Output:
```

OUTPUT:
```markdown
Alright! Let's start by examining the contents of the CSV file to understand its structure. I will read the first few rows of the file.

import pandas as pd

# Load the CSV file
df = pd.read_csv('/mnt/data/input_sentences.csv')

# Display the first few rows
df.head()


The CSV file contains a column named "post" with sentences. I will now create a new text file and format each sentence from the "post" column as:

```
Input: <input_sentence>
Output:
```

Let's generate the output text file.

# Create the formatted text for each sentence
formatted_text = ""
for sentence in df['post']:
    formatted_text += f"Input: {sentence}\nOutput:\n\n"

# Write the formatted text to a new output file
output_file_path = "/mnt/data/transformed_sentences.txt"
with open(output_file_path, 'w') as f:
    f.write(formatted_text)

output_file_path


I have transformed the sentences and written them to a new text file. You can [download the transformed sentences here](sandbox:/mnt/data/transformed_sentences.txt).
```

I saved this file to `transformed_sentences.txt`

## Step 3 - Run annotation Prompt

I used the `transformed_sentences.txt` file create a prompt I can use to annotate my data, using the example sentences with topics.

Here's the prompt:

PROMPT (Excerpt)

```markdown
Fill in the output

Input: The athlete won a gold medal at the Olympics.
Output: sports

Input: The legislation was passed after a long debate in the Senate.
Output: politics

Input: The discovery of the Higgs boson at CERN marked a milestone in particle physics.
Output: science

Input: The swimmer broke the world record in the 100-meter freestyle.
Output: sports

Input: The bill received bipartisan support and was signed into law by the president.
Output: politics

Input: The recent findings on black holes have revolutionized our understanding of space-time.
Output: science

Input: The spacecraft successfully docked with the International Space Station.
Output: science

Input: The gymnast captured the audience's hearts and took home the bronze medal.
Output: sports

Input: After months of negotiations, the climate accord was finally agreed upon by the majority of nations.
Output: politics

Input: The marathon runner achieved a personal best in the city marathon.
Output:

Input: The amendment to the constitution was approved after rigorous discussions in the assembly.
Output:

...
```

OUTPUT (excerpt):

```markdown
Input: The marathon runner achieved a personal best in the city marathon.
Output: sports

Input: The amendment to the constitution was approved after rigorous discussions in the assembly.
Output: politics

Input: The detection of gravitational waves opened a new chapter in astrophysics.
Output: science

Input: The sprinter set a new national record in the 200-meter dash.
Output: sports

Input: The proposed act gained unanimous approval and was ratified by the governor.
Output: politics

Input: New research on dark matter has altered our comprehension of the universe.
Output: science

Input: The satellite was launched into orbit without any hitches.
Output: science

Input: The diver impressed the judges and secured a silver medal.
Output: sports

...
```

I saved the output to a file: `annotated_sentences.txt`

## Step 4 - Format as a csv

I asked ChatGPT's advanced analysis mode to run a script to transform `annotated_sentences.txt` to a new csv file.

PROMPT:

```markdown
Run a script that takes the text file as input and outputs a new csv file.

The text file has multiple entries like this:

Input: The scientist presented his findings at the international conference.
Output: science

Transform to a csv file, mapping Input to column post and Output to column topic
post,topic
The scientist presented his findings at the international conference.,science
```

Code interpreter had some back and forth to figure out the exact logic, but came up with this code:

```python
import csv

# Read the text file
with open("/mnt/data/annotated_sentences.txt", "r") as file:
    lines = file.readlines()

# Parse the lines to extract Input and Output values
data = []
i = 0
while i < len(lines):
    if lines[i].startswith("Input:"):
        input_line = lines[i].split(": ")[1].strip()
        i += 1
        if i < len(lines) and lines[i].startswith("Output:"):
            output_line = lines[i].split(": ")[1].strip()
            data.append((input_line, output_line))
            i += 1
        else:
            # Skip until next Input if Output is missing
            while i < len(lines) and not lines[i].startswith("Input:"):
                i += 1
    else:
        i += 1

# Write the processed data to a csv file
csv_filename = "/mnt/data/processed_data.csv"
with open(csv_filename, "w", newline='') as file:
    writer = csv.writer(file)
    writer.writerow(["post", "topic"])
    writer.writerows(data)

```

Maybe not a clean as I would have written it, but it works! I downloaded the result to `final_results.csv`


## Observation

To me the process outlined in the exercise seemed a little overly complex. I don't think we actually need to convert the annotations to a prompt in text format first. I think you could do all of this with a single prompt, giving your few-shot examples in a csv format. I tried with a prompt like this:

PROMPT:

```
Here's an example csv file with a news post, classified with a topic. Create a new csv with 90 similar posts and topics

post,topic
The athlete won a gold medal at the Olympics.,sports
The legislation was passed after a long debate in the Senate.,politics
The discovery of the Higgs boson at CERN marked a milestone in particle physics.,science
The swimmer broke the world record in the 100-meter freestyle.,sports
The bill received bipartisan support and was signed into law by the president.,politics
The recent findings on black holes have revolutionized our understanding of space-time.,science
The spacecraft successfully docked with the International Space Station.,science
The gymnast captured the audience's hearts and took home the bronze medal.,sports
"After months of negotiations, the climate accord was finally agreed upon by the majority of nations.",politics
"Scientists successfully cloned a sheep, sparking ethical debates worldwide.",science
```

This creates a csv file just fine:

OUTPUT (Excerpt):

```markdown
Certainly, here's an expanded list of similar news posts with corresponding topics:


post,topic
The athlete won a gold medal at the Olympics.,sports
The legislation was passed after a long debate in the Senate.,politics
The discovery of the Higgs boson at CERN marked a milestone in particle physics.,science
The swimmer broke the world record in the 100-meter freestyle.,sports
The bill received bipartisan support and was signed into law by the president.,politics
```

This was with the GPT-4 model. It might be needed to break to process into steps with less powerful models like GPT-3.5, but GPT-4 can do it all in one step.


# Task 2 - Adverserial Examples

## Step 1 - Asking ChatGPT

I asked the LLM if it could help me think of examples.

PROMPT:

```markdown
Let's say I was prompting a Language model to classify sentences like in these examples

Input: The athlete won a gold medal at the Olympics.
Output: sports

Input: The legislation was passed after a long debate in the Senate.
Output: politics

Input: The discovery of the Higgs boson at CERN marked a milestone in particle physics.
Output: science

Input: The swimmer broke the world record in the 100-meter freestyle.
Output: sports

Input: The bill received bipartisan support and was signed into law by the president.
Output: politics

Input: The recent findings on black holes have revolutionized our understanding of space-time.
Output: science

Input: The spacecraft successfully docked with the International Space Station.
Output: science

Input: The gymnast captured the audience's hearts and took home the bronze medal.
Output: sports

Input: After months of negotiations, the climate accord was finally agreed upon by the majority of nations.
Output: politics

Input: "Scientists successfully cloned a sheep, sparking ethical debates worldwide."
Output: science

Could you come up with some good example sentences it might get wrong?
```

OUTPUT:

```

Certainly! Ambiguities often arise when sentences contain elements from multiple categories, or when they reference concepts that might be associated with more than one category. Here are some examples:

Input: The senator ran a marathon to raise awareness for climate change.
Expected Output: politics
AI might guess: sports

Input: The physicist was elected as the new mayor of the city.
Expected Output: politics
AI might guess: science

Input: The international chess championship became a political battleground between two countries.
Expected Output: politics
AI might guess: sports

Input: The scientist delivered a passionate speech at the political rally.
Expected Output: politics
AI might guess: science

Input: The new football stadium became a hub for renewable energy research.
Expected Output: science
AI might guess: sports

Input: The astronaut competed in Dancing with the Stars.
Expected Output: sports (or entertainment, if that were a category)
AI might guess: science

Input: The policy was based on the most recent geological findings in the region.
Expected Output: politics
AI might guess: science

Input: The political debate reached a peak when both candidates discussed their favorite workout routines.
Expected Output: politics
AI might guess: sports

Input: The biathlete discussed the impacts of global warming on snow conditions.
Expected Output: science
AI might guess: sports

Input: The world record for the most politicians in a tug-of-war match was broken last summer.
Expected Output: politics
AI might guess: sports

These examples are crafted to be challenging and ambiguous by blending elements from different categories. In practice, context and deeper understanding are required to classify them correctly.
```

## Step 2 - Exploit ambiguity and the knowledge cutoff

These all seemed like they would be ambiguous to a human as well. But that gave me an idea. I could probably use a combination of this idea of ambiguity, but also exploit the LLM's knowledge cutoff weakness to come up with examples of things that human's would probably be able to classify correctly but not LLM's.

For example, this prompt:

PROMPT
```markdown

post,topic
The athlete won a gold medal at the Olympics.,sports
The legislation was passed after a long debate in the Senate.,politics
The discovery of the Higgs boson at CERN marked a milestone in particle physics.,science
The swimmer broke the world record in the 100-meter freestyle.,sports
The bill received bipartisan support and was signed into law by the president.,politics
The recent findings on black holes have revolutionized our understanding of space-time.,science
The spacecraft successfully docked with the International Space Station.,science
The gymnast captured the audience's hearts and took home the bronze medal.,sports
"After months of negotiations, the climate accord was finally agreed upon by the majority of nations.",politics
"Scientists successfully cloned a sheep, sparking ethical debates worldwide.",science
Vivek Ramaswamy scored a touchdown last week.,
```

OUTPUT:

```markdown
sports
```

The LLM guesses that topic is sports because of the term 'touchdown' being used and associating that with football. A human that's been following more recent US news would probably classify this a being about politics because it references a polititian [Vivek Ramaswamy](https://en.wikipedia.org/wiki/Vivek_Ramaswamy) who's only been in the news fairly recently because of his presidential candidacy.

At first I thought because of the LLM's knowledge cutoff, it doesn't know that Vivek Ramaswamy is a polititian, the ambiguity of the sports term throws it off.

I tested this hypotheis by trying a similar prompt, but instead referencing Joe Biden, who's a much more well known political figure.

PROMPT:

```markdown
post,topic
The athlete won a gold medal at the Olympics.,sports
The legislation was passed after a long debate in the Senate.,politics
The discovery of the Higgs boson at CERN marked a milestone in particle physics.,science
The swimmer broke the world record in the 100-meter freestyle.,sports
The bill received bipartisan support and was signed into law by the president.,politics
The recent findings on black holes have revolutionized our understanding of space-time.,science
The spacecraft successfully docked with the International Space Station.,science
The gymnast captured the audience's hearts and took home the bronze medal.,sports
"After months of negotiations, the climate accord was finally agreed upon by the majority of nations.",politics
"Scientists successfully cloned a sheep, sparking ethical debates worldwide.",science
Joe Biden scored a touchdown last week.,
```

OUTPUT:

```markdown
sports
```

It still gets it wrong! So it turned out I didn't even need to be as clever with exploiting the knowledge cutoff, just using a ambiguous term works.

You can probably come up with many more examples that follow this pattern.
