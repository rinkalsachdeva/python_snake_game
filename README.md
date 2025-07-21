import pandas as pd
import matplotlib.pyplot as plt
import math
import os
import textwrap

# Load CSV
csv_file = 'new_data.csv'
df = pd.read_csv(csv_file)

# Remove columns that are completely blank (NaN or empty)
df = df.dropna(axis=1, how='all')  # Drops all-empty columns

# Output directory
output_dir = 'screenshots'
os.makedirs(output_dir, exist_ok=True)

# Dynamic text wrapping function
def wrap_and_split(text, max_line_chars=40):
    """Wrap text aggressively for URLs and long text."""
    if pd.isna(text):
        return ""
    text = str(text)
    return "\n".join(textwrap.wrap(text, width=max_line_chars))

# Apply wrapping to all cells
wrapped_df = df.applymap(wrap_and_split)

# Split into groups of 10 rows per image
rows_per_image = 10
total_chunks = math.ceil(len(wrapped_df) / rows_per_image)

for i in range(total_chunks):
    start = i * rows_per_image
    end = start + rows_per_image
    chunk = wrapped_df.iloc[start:end]

    # Adjust figure size dynamically based on content
    max_lines = chunk.applymap(lambda x: str(x).count("\n") + 1).max().max()
    fig_height = 3 + (0.6 * rows_per_image * max_lines)
    fig_width = 20  # Adjust width for long links

    # Create figure
    fig, ax = plt.subplots(figsize=(fig_width, fig_height))
    ax.axis('off')

    # Create the table
    table = ax.table(cellText=chunk.values,
                     colLabels=chunk.columns,
                     cellLoc='center',
                     loc='center')

    # Styling
    table.auto_set_font_size(False)
    table.set_fontsize(9)
    table.scale(1.5, 2.0)

    # Style header row (bold, yellow)
    for (row, col), cell in table.get_celld().items():
        if row == 0:
            cell.set_text_props(weight='bold', color='black')
            cell.set_facecolor('#FFD700')
        else:
            cell.set_facecolor('white')

    # Save the screenshot
    image_path = os.path.join(output_dir, f'screenshot_{i+1}.png')
    plt.savefig(image_path, bbox_inches='tight', dpi=200)
    plt.close()

print("Screenshots created with blank columns removed!")
