1. Import Libraries:

import random
import tkinter as tk
from tkinter import filedialog
from PIL import Image, ImageTk
random: Used to shuffle the tiles.
tkinter: Python's standard GUI (Graphical User Interface) library.
filedialog: A module from tkinter that allows users to select files from their system.
PIL (Pillow): A library for image processing in Python. Image is used for handling images, and ImageTk converts PIL images into formats usable in tkinter.
2. Class Definition (ImagePuzzle):

class ImagePuzzle:
This defines a class ImagePuzzle that will encapsulate all the logic for creating and managing the puzzle game.

3. Initialize the Class (__init__):

def __init__(self, root):
    self.root = root
    self.root.title("Sliding Image Puzzle Game")
    self.grid_size = 3  # Default grid size (3x3)
    self.tile_size = None
    self.tiles = []
    self.empty_tile = None
    self.original_image = None
    self.puzzle_frame = tk.Frame(root)
    self.puzzle_frame.grid(row=1, column=0, columnspan=4)

    self.load_button = tk.Button(root, text="Load Image", command=self.load_image)
    self.load_button.grid(row=0, column=0)

    self.shuffle_button = tk.Button(root, text="Shuffle", command=self.shuffle_tiles)
    self.shuffle_button.grid(row=0, column=1)

    self.reset_button = tk.Button(root, text="Reset", command=self.reset_puzzle)
    self.reset_button.grid(row=0, column=2)

    self.preview_button = tk.Button(root, text="Preview", command=self.show_preview)
    self.preview_button.grid(row=0, column=3)
self.root = root: Binds the tkinter root window to the ImagePuzzle object.
self.root.title: Sets the title of the window to "Sliding Image Puzzle Game".
self.grid_size = 3: Initializes a 3x3 grid (i.e., a 9-tile puzzle).
self.tile_size, self.tiles, self.empty_tile, self.original_image: Various attributes that are initialized but will be set later when the image is loaded.
self.puzzle_frame: Creates a Frame widget (like a container) where the puzzle tiles will be displayed.
grid(row=1, column=0, columnspan=4): Places self.puzzle_frame in the GUI layout, spanning across 4 columns.
Next, buttons are created:

Load Image Button: Clicking this button calls load_image() to load an image file.
Shuffle Button: Calls shuffle_tiles() to randomize the tile positions.
Reset Button: Calls reset_puzzle() to reset the puzzle to its original state.
Preview Button: Calls show_preview() to show the full original image in a new window.
4. Loading an Image (load_image):

def load_image(self):
    file_path = filedialog.askopenfilename()
    if file_path:
        self.original_image = Image.open(file_path)
        self.create_puzzle()
filedialog.askopenfilename(): Opens a file selection dialog, allowing the user to choose an image.
Image.open(file_path): Loads the selected image using PIL's Image.open.
self.create_puzzle(): Calls the create_puzzle method to break the image into smaller pieces and set up the puzzle grid.
5. Creating the Puzzle Grid (create_puzzle):

def create_puzzle(self):
    self.tile_size = (self.original_image.width // self.grid_size, self.original_image.height // self.grid_size)

    self.tiles.clear()
    for row in range(self.grid_size):
        row_tiles = []
        for col in range(self.grid_size):
            if row == self.grid_size - 1 and col == self.grid_size - 1:
                row_tiles.append(None)  # Empty tile
            else:
                left = col * self.tile_size[0]
                top = row * self.tile_size[1]
                right = left + self.tile_size[0]
                bottom = top + self.tile_size[1]
                tile_image = self.original_image.crop((left, top, right, bottom))
                row_tiles.append(tile_image)
        self.tiles.append(row_tiles)

    self.empty_tile = (self.grid_size - 1, self.grid_size - 1)  # Keep track of empty tile position
    self.update_puzzle_display()
self.tile_size: Sets each tile's size as the full image dimensions divided by the grid size.
self.tiles.clear(): Clears any previously stored tiles.
for row in range(self.grid_size): Iterates over each row and column in the grid to break the image into smaller tiles using crop().
self.original_image.crop(): Extracts a rectangular tile from the original image.
row_tiles.append(None): The last tile is left empty (i.e., the puzzle's "blank" space).
self.tiles.append(row_tiles): Adds each row of tiles to the overall grid.
self.empty_tile: Stores the position of the empty tile.
6. Displaying the Puzzle Grid (update_puzzle_display):

def update_puzzle_display(self):
    for widget in self.puzzle_frame.winfo_children():
        widget.destroy()

    for row in range(self.grid_size):
        for col in range(self.grid_size):
            if self.tiles[row][col] is not None:
                img = ImageTk.PhotoImage(self.tiles[row][col])
                tile_button = tk.Button(self.puzzle_frame, image=img, command=lambda r=row, c=col: self.move_tile(r, c))
                tile_button.grid(row=row, column=col)
                tile_button.image = img  # Keep reference to avoid garbage collection
            else:
                tile_button = tk.Button(self.puzzle_frame, bg='black', state='disabled')
                tile_button.grid(row=row, column=col)
self.puzzle_frame.winfo_children(): Gets all child widgets (i.e., buttons) from the frame, and widget.destroy() removes them before re-drawing the updated puzzle.
for row, col in grid: Adds a button for each tile. For non-empty tiles, the button shows the image and allows swapping tiles via move_tile(). For the empty tile, it creates a blank button.
tile_button.image = img: Ensures the image isn't garbage collected by storing a reference to it.
7. Shuffling the Tiles (shuffle_tiles):

def shuffle_tiles(self):
    moves = [(0, 1), (0, -1), (1, 0), (-1, 0)]
    empty_row, empty_col = self.empty_tile

    for _ in range(100):
        valid_moves = []
        for move in moves:
            new_row, new_col = empty_row + move[0], empty_col + move[1]
            if 0 <= new_row < self.grid_size and 0 <= new_col < self.grid_size:
                valid_moves.append((new_row, new_col))

        if valid_moves:
            target_row, target_col = random.choice(valid_moves)
            self.tiles[empty_row][empty_col], self.tiles[target_row][target_col] = self.tiles[target_row][target_col], self.tiles[empty_row][empty_col]
            empty_row, empty_col = target_row, target_col

    self.empty_tile = (empty_row, empty_col)
    self.update_puzzle_display()
moves: Defines possible directions for moving the tiles (right, left, down, up).
valid_moves: For each iteration, checks which moves are valid by ensuring they stay within the grid bounds.
random.choice(valid_moves): Picks a random valid move and swaps the empty tile with a neighboring one.
The loop runs 100 times, making the puzzle sufficiently shuffled.
8. Moving a Tile (move_tile):

def move_tile(self, row, col):
    empty_row, empty_col = self.empty_tile

    if (abs(empty_row - row) == 1 and empty_col == col) or (abs(empty_col - col) == 1 and empty_row == row):
        self.tiles[empty_row][empty_col], self.tiles[row][col] = self.tiles[row][col], self.tiles[empty_row][empty_col]
        self.empty_tile = (row, col)
        self.update_puzzle_display()
Checks adjacency: If the selected tile is adjacent to the empty space, it swaps them.
9. Reset Puzzle (reset_puzzle):

def reset_puzzle(self):
    self.create_puzzle()
Calls create_puzzle() to restore the original tile arrangement.
10. Preview the Original Image (show_preview):

def show_preview(self):
    if self.original_image:
        preview_window = tk.Toplevel(self.root)
        preview_window.title("Image Preview")

        preview_image = self.original_image.resize((300, 300), Image.Resampling.LANCZOS)
        img = ImageTk.PhotoImage(preview_image)
        
       










#   i m a g e - s h u f f e l  
 