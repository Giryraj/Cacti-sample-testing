import os
import shutil

def find_and_copy_files(search_folder, partial_names, destination_folder):
    """
    Checks if files with partial names exist in a folder or its subfolders,
    and copies them to a new directory.

    Args:
        search_folder (str): The path to the folder to search within.
        partial_names (list): A list of strings representing partial file names to search for.
        destination_folder (str): The path to the directory where matching files will be copied.
    """
    if not os.path.exists(destination_folder):
        os.makedirs(destination_folder)
        print(f"Created destination folder: {destination_folder}")

    found_count = 0
    copied_count = 0

    for root, _, files in os.walk(search_folder):
        for filename in files:
            for partial_name in partial_names:
                if partial_name.lower() in filename.lower():
                    source_path = os.path.join(root, filename)
                    destination_path = os.path.join(destination_folder, filename)

                    try:
                        shutil.copy2(source_path, destination_path)  # copy2 preserves metadata
                        print(f"Copied: {source_path} to {destination_path}")
                        copied_count += 1
                        found_count += 1
                        # Once a match is found and copied, no need to check other partial names for this file
                        break
                    except Exception as e:
                        print(f"Error copying {source_path}: {e}")
                    break  # Break out of the partial_names loop if a match is found

    print(f"\nSearched in: {search_folder}")
    print(f"Found {found_count} files matching the partial names.")
    print(f"Successfully copied {copied_count} files to: {destination_folder}")

if __name__ == "__main__":
    search_directory = "/path/to/your/search/folder"  # Replace with the actual search folder path
    partial_file_names = ["report_", "image-", "data_2023"]  # Example partial file names
    copy_to_directory = "/path/to/your/destination/folder"  # Replace with the desired destination folder path

    find_and_copy_files(search_directory, partial_file_names, copy_to_directory)

Explanation:
 * Import necessary modules:
   * os: Provides functions for interacting with the operating system, such as walking through directories and checking if paths exist.
   * shutil: Offers high-level file operations, including copying files.
 * find_and_copy_files function:
   * Takes three arguments:
     * search_folder: The root directory where the search will begin.
     * partial_names: A list of strings. The code will look for files whose names contain any of these strings (case-insensitive).
     * destination_folder: The directory where the found files will be copied.
   * Creates destination folder (if it doesn't exist):
     * if not os.path.exists(destination_folder): checks if the destination folder exists.
     * os.makedirs(destination_folder) creates the folder and any necessary parent directories.
   * Initializes counters:
     * found_count and copied_count are used to track the number of matching files found and successfully copied.
   * os.walk(search_folder):
     * This function is a generator that yields tuples for each directory it encounters within search_folder (including subdirectories). Each tuple contains:
       * root: The path to the current directory.
       * dirs: A list of subdirectory names in the current directory.
       * files: A list of file names in the current directory.
   * Iterating through files:
     * The code iterates through each filename in the files list of the current directory.
   * Checking for partial names:
     * For each filename, it iterates through the partial_names list.
     * if partial_name.lower() in filename.lower(): checks if the lowercase version of the partial_name is present in the lowercase version of the filename. This ensures a case-insensitive search.
   * Constructing file paths:
     * source_path = os.path.join(root, filename) creates the full path to the source file.
     * destination_path = os.path.join(destination_folder, filename) creates the full path for the copied file in the destination folder.
   * Copying the file:
     * shutil.copy2(source_path, destination_path) copies the file from the source to the destination. copy2 is used to preserve as much metadata as possible (like timestamps).
     * A try-except block is used to handle potential errors during the copying process (e.g., permission issues).
   * Incrementing counters and breaking loops:
     * If a file is successfully copied, copied_count and found_count are incremented.
     * break is used to exit the inner loop (for partial_name in partial_names:) once a match is found for a particular file. This prevents the same file from being copied multiple times if it matches multiple partial names.
   * Printing summary:
     * After traversing all directories and files, the function prints a summary of the search and copy operations.
 * if __name__ == "__main__": block:
   * This block ensures that the code inside it only runs when the script is executed directly (not when it's imported as a module).
   * Sets example variables:
     * search_directory: You need to replace this with the actual path to the folder you want to search.
     * partial_file_names: A list of example partial file names. Modify this list with the partial names you are looking for.
     * copy_to_directory: You need to replace this with the actual path to the directory where you want to copy the files.
   * Calls the find_and_copy_files function with the defined variables.
How to Use:
 * Save the code: Save the Python code as a .py file (e.g., find_copy.py).
 * Modify the paths and partial names:
   * Replace /path/to/your/search/folder with the actual path to the folder you want to search within.
   * Modify the partial_file_names list with the partial strings you want to use for searching. For example, if you want to find files containing "report" or "image", your list would be ["report", "image"].
   * Replace /path/to/your/destination/folder with the actual path to the folder where you want the matching files to be copied.
 * Run the script: Open a terminal or command prompt, navigate to the directory where you saved the file, and run it using:
   python find_copy.py

The script will then search the specified folder and its subfolders for files containing the given partial names and copy them to the destination folder. You will see output indicating which files were found and copied.
