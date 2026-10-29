# Scientific Computing with NumPy

## Computation with Base Python

At its core, Python is a general-purpose language that fully supports standard arithmetic operations.

```python
# Standard arithmetic
a = 15.5
b = 4
print(f"a + b = {a + b}") # Addition
print(f"a / b = {a / b}") # Floating-point division
print(f"a // b = {a // b}") # Integer (floor) division

# Exponentiation and Modulo
print(f"2 to the power of 10 is {2**10}")
print(f"17 modulo 3 is {17 % 3}")

# Python's integers have arbitrary precision
large_number = 2**1024
print(f"A very large number: {large_number}")
```

This is sufficient for simple, scalar (single-value) calculations. However, scientific computing rarely deals with single numbers; it deals with large collections of them. The default collection in Python is the `list`.

## `math` Module

For anything beyond basic arithmetic, the standard library provides the `math` module. This module gives us access to fundamental trigonometric, logarithmic, and exponential functions, as well as important constants.

```python
import math

# Constants
print(f"The value of Pi: {math.pi}")
print(f"The value of Euler's number (e): {math.e}")

# Common functions
angle_radians = math.pi / 4 # 45 degrees
print(f"Sine of 45 degrees: {math.sin(angle_radians):.4f}")
print(f"Square root of 81: {math.sqrt(81)}")
print(f"Natural logarithm of 10: {math.log(10):.4f}")
```

**Important Note:** Functions in the `math` module are designed to operate on **single scalar values**, not on lists or other collections of numbers. This is a critical limitation.

### Example: Projectile Motion

Let's model a simple physical system: the trajectory of a projectile (e.g., a ball) thrown at an angle. The formulas for its position `(x, y)` over time `t` are:

*   `x(t) = v₀ * t * cos(θ)`
*   `y(t) = v₀ * t * sin(θ) - 0.5 * g * t²`

Where:

*   `v₀` is the initial velocity.
*   `θ` is the launch angle.
*   `g` is the acceleration due to gravity (approx. 9.81 m/s²).

Let's calculate the trajectory at 1,000 time steps using Python lists and the `math` module.

```python
import math

def calculate_trajectory(v0, theta_deg, num_steps=1000):
    """Calculates the x, y coordinates of a projectile over time."""
    g = 9.81
    theta_rad = math.radians(theta_deg)
    
    # Find the total time of flight to scale our time steps
    total_time = (2 * v0 * math.sin(theta_rad)) / g
    
    # Generate time steps
    time_steps = [total_time * i / num_steps for i in range(num_steps)]
    
    x_coords = []
    y_coords = []
    
    for t in time_steps:
        x = v0 * t * math.cos(theta_rad)
        y = v0 * t * math.sin(theta_rad) - 0.5 * g * t**2
        x_coords.append(x)
        y_coords.append(y)
        
    return x_coords, y_coords

# Run the simulation
v_initial = 50  # m/s
angle = 45      # degrees
x_traj, y_traj = calculate_trajectory(v_initial, angle)

print(f"Calculated {len(x_traj)} points.")
print(f"Final X position: {x_traj[-1]:.2f}m")
```

This code works perfectly fine. The logic is clear and it produces the correct result. So, what's the problem?

### Performance Bottleneck

The problem is **performance**. Python is a dynamically-typed, interpreted language. When the `for` loop in our function runs, for each number in `time_steps`, Python must:

1.  Check the type of the number.
2.  Look up which specific multiplication/subtraction function to call for that type.
3.  Perform the operation.
4.  Create a new Python object for the result.

This overhead is negligible for a few operations, but it becomes a major bottleneck when dealing with thousands or millions of data points, which is common in scientific applications.

Let's measure it. We will compare a pure Python loop for adding two lists of numbers against the equivalent operation in NumPy.

```python
import time

# Create two lists with one million numbers each
list_a = list(range(1_000_000))
list_b = list(range(1_000_000))

# --- Pure Python approach ---
start_time = time.time()
result_list = []
for i in range(len(list_a)):
    result_list.append(list_a[i] + list_b[i])
end_time = time.time()
print(f"Pure Python loop took: {end_time - start_time:.4f} seconds")

# --- Preview of the NumPy approach ---
import numpy as np
array_a = np.arange(1_000_000)
array_b = np.arange(1_000_000)

start_time = time.time()
result_array = array_a + array_b
end_time = time.time()
print(f"NumPy vectorized operation took: {end_time - start_time:.4f} seconds")
```

You will see a dramatic difference in performance—often 50-100x faster for the NumPy version. This is why for any serious numerical work, we leave base Python behind and turn to specialized libraries.

## Introduction to NumPy

**NumPy (Numerical Python)** is the fundamental package for scientific computing in Python. Its core feature is a powerful N-dimensional array object called the `ndarray`.

**Why is the `ndarray` so much better than a Python list?**

*   **Homogeneous Type:** All elements in an array must be of the same data type (e.g., all `float64` or all `int32`). This eliminates the type-checking overhead of Python lists.
*   **Contiguous Memory:** The elements are stored in an unbroken block of memory. This allows for extremely fast access and manipulation by optimized, pre-compiled C or Fortran code that runs "under the hood."
*   **Vectorization:** NumPy allows you to perform operations on entire arrays at once, without writing explicit `for` loops in Python. This is called **vectorization**.

First, let's install it and adopt the conventional alias `np`.

```shell
pip install numpy
```

```python
import numpy as np
```

#### Creating NumPy Arrays

```python
# From a Python list
my_list = [1, 2, 3, 4, 5]
my_array = np.array(my_list)
print(f"1D Array: {my_array}")
print(f"Type: {my_array.dtype}") # e.g., int64

# Creating arrays from scratch
zeros = np.zeros((2, 3)) # A 2x3 matrix of zeros
print(f"\nZeros:\n{zeros}")

ones = np.ones(5) # A 1D array of five ones
print(f"\nOnes: {ones}")

# Creating sequences
sequence = np.arange(0, 10, 2) # Start, stop (exclusive), step
print(f"\nSequence (arange): {sequence}")

evenly_spaced = np.linspace(0, 1, 5) # Start, stop (inclusive), num points
print(f"\nEvenly spaced (linspace): {evenly_spaced}")
```

## Vectorization

Vectorization is the process of applying an operation to an entire array at once, rather than element-by-element. This delegates the looping to NumPy's highly optimized, pre-compiled C code. The Python code becomes cleaner, more readable, and orders of magnitude faster.

Let's rewrite our projectile motion simulation using NumPy.

```python
import numpy as np

def calculate_trajectory_numpy(v0, theta_deg, num_steps=1000):
    """Calculates the projectile trajectory using NumPy vectorization."""
    g = 9.81
    theta_rad = np.radians(theta_deg) # NumPy has its own math functions
    
    total_time = (2 * v0 * np.sin(theta_rad)) / g
    
    # Create an array of time steps
    t = np.linspace(0, total_time, num_steps)
    
    # Perform vectorized calculations
    x = v0 * t * np.cos(theta_rad)
    y = v0 * t * np.sin(theta_rad) - 0.5 * g * t**2
    
    return x, y

# Run the simulation
x_traj_np, y_traj_np = calculate_trajectory_numpy(v_initial, angle)

print(f"Calculated {len(x_traj_np)} points using NumPy.")
print(f"Final X position: {x_traj_np[-1]:.2f}m")
```

Notice how the explicit `for` loop has vanished. The code is not only faster but also more concise and closer to the original mathematical formulas.

## Advanced Array Operations

#### Indexing and Slicing
NumPy extends Python's slicing syntax to multiple dimensions.

```python
# Create a 3x4 array
data = np.array([[1, 2, 3, 4],
                 [5, 6, 7, 8],
                 [9, 10, 11, 12]])

# Get a single element (row 1, column 2)
element = data[1, 2] # Result: 7
print(f"Element at [1, 2]: {element}")

# Get a row
row_1 = data[1, :] # Result: [5, 6, 7, 8]
print(f"Row 1: {row_1}")

# Get a column
col_2 = data[:, 2] # Result: [3, 7, 11]
print(f"Column 2: {col_2}")

# Get a sub-matrix (top-left 2x2)
sub_matrix = data[:2, :2]
print(f"\nSub-matrix:\n{sub_matrix}")
```

#### Boolean Indexing
Just like with Pandas, we can use boolean arrays to filter data.

**Example: Image Processing**
A grayscale image can be represented as a 2D NumPy array where each value is a pixel intensity (0=black, 255=white).

```python
# Create a dummy 5x5 'image' with random pixel values
image = np.random.randint(0, 256, size=(5, 5))
print(f"Original 'image':\n{image}")

# Find all pixels that are 'bright' (value > 150)
bright_pixels = image > 150
print(f"\nBoolean mask for bright pixels:\n{bright_pixels}")

# Use the mask to get the actual values of bright pixels
print(f"\nValues of bright pixels: {image[bright_pixels]}")

# We can also use it to modify the image, e.g., make all bright pixels white
image[bright_pixels] = 255
print(f"\nModified 'image' (thresholded):\n{image}")
```

#### Array Shape Manipulation

You can change the dimensions of an array as long as the total number of elements remains the same.

```python
# Create a 1D array with 12 elements
a = np.arange(12)
print(f"Original array: {a}")

# Reshape it into a 3x4 matrix
b = a.reshape(3, 4)
print(f"\nReshaped 3x4 array:\n{b}")

# The total size must be conserved. This would raise an error:
# a.reshape(3, 5) # ValueError: cannot reshape array of size 12 into shape (3,5)
```

The reverse of reshaping is flattening; this operation collapses a multi-dimensional array into a single dimension.

*   `.flatten()`: Always returns a new copy of the data.
*   `.ravel()`: Returns a "view" of the original array whenever possible, meaning it doesn't use extra memory. Changes to the raveled array might affect the original array.

```python
# Using the 3x4 matrix 'b' from before
flat_copy = b.flatten()
print(f"Flattened (copy): {flat_copy}")

ravel_view = b.ravel()
print(f"Raveled (view): {ravel_view}")

# Modifying the raveled array may change the original matrix 'b'
ravel_view[0] = 99
print(f"\nOriginal matrix 'b' after modifying the raveled view:\n{b}")
```

Transposing an array swaps its axes. For a 2D matrix, this flips it over its diagonal (rows become columns and columns become rows).

```python
print(f"Original matrix b (3x4):\n{b}")

transposed = b.T
print(f"\nTransposed matrix (4x3):\n{transposed}")
```

#### Linear Algebra

- **Element-wise multiplication (`*`)**: The arrays must have compatible shapes (or be broadcastable). The operation is performed on elements in the same position.
- **Matrix multiplication (`@` or `np.dot()`)**: This performs the linear algebra dot product. The inner dimensions of the matrices must match (e.g., a `(M x N)` matrix can be multiplied by a `(N x K)` matrix).

```python
mat_A = np.array([[1, 2], [3, 4]])
mat_B = np.array([[5, 6], [7, 8]])

# Element-wise multiplication
element_wise_prod = mat_A * mat_B
print(f"Element-wise product:\n{element_wise_prod}")
# Result is [[1*5, 2*6], [3*7, 4*8]]

# Matrix multiplication (dot product)
# Using the @ operator (recommended in modern Python)
matrix_prod = mat_A @ mat_B
print(f"\nMatrix product (using @):\n{matrix_prod}")

# Using np.dot()
matrix_prod_dot = np.dot(mat_A, mat_B)
print(f"\nMatrix product (using np.dot):\n{matrix_prod_dot}")
# Result follows matrix multiplication rules:
# [[1*5+2*7, 1*6+2*8], [3*5+4*7, 3*6+4*8]]
```

## Generating Random Data

In simulation, statistics, and machine learning, we constantly need to generate random numbers. It's important to understand what "random" means in the context of a deterministic machine like a computer.

#### Randomness vs. Pseudo-Randomness

*   **True Randomness**: Sourced from unpredictable physical phenomena like atmospheric noise or radioactive decay. It is non-deterministic and non-reproducible.
*   **Pseudo-Randomness**: This is what computers use. A **pseudo-random number generator (PRNG)** is an algorithm that produces a sequence of numbers that approximates the properties of random numbers. The sequence is determined by an initial value called a **seed**.

#### The Importance of the Seed

The **seed** is the starting point for the PRNG. If you provide the same seed, you will get the exact same sequence of "random" numbers every single time. It ensures **reproducibility**, which is essential for:

*   **Debugging:** You can re-run your code with the same random numbers to track down errors.
*   **Scientific Experiments:** Others can reproduce your results exactly.
*   **Fair Comparisons:** You can test different algorithms on the same "random" dataset.

If you don't set a seed, one is usually chosen for you based on the current system time, making the results different on each run.

#### Python's `random` Module

This module is part of the standard library and is great for simple, scalar random numbers.

```python
import random

# Set a seed for reproducibility
random.seed(42)

# Generate a random float between 0.0 and 1.0
print(f"Random float: {random.random()}")

# Generate a random integer between 1 and 10 (inclusive)
print(f"Random integer: {random.randint(1, 10)}")

# Running it again with the same seed will produce the same output
random.seed(42)
print(f"\nRandom float (same seed): {random.random()}")
print(f"Random integer (same seed): {random.randint(1, 10)}")
```

#### NumPy's `random` Module

NumPy's `random` module is highly optimized for the generation of pseudo-random numbers. The modern, recommended approach is to create a `Generator` instance.

```python
from numpy.random import default_rng

# Create a generator instance with a seed
rng = default_rng(seed=42)

# Generate a 3x4 array of random floats from a uniform distribution [0.0, 1.0)
uniform_array = rng.random((3, 4))
print(f"Array of uniform random floats:\n{uniform_array}")

# Generate a 3x4 array of random floats from a standard normal distribution
# (mean=0, variance=1)
normal_array = rng.standard_normal((3, 4))
print(f"\nArray of standard normal random floats:\n{normal_array}")

# Generate a 1D array of 10 random integers between 0 (inclusive) and 100 (exclusive)
int_array = rng.integers(low=0, high=100, size=10)
print(f"\nArray of random integers: {int_array}")
```

Using NumPy's random generation is vastly more efficient than generating numbers one by one in a Python loop and appending them to a list.

## Beyond NumPy

NumPy provides the fundamental array object and basic operations. However, the scientific ecosystem is built on layers, with NumPy at the bottom.

#### `SciPy` (Scientific Python)
If NumPy provides the data structure (`ndarray`), **SciPy provides the algorithms**. It's a library of modules for performing common scientific and technical computing tasks. It is particularly useful for things like:

*   **`scipy.optimize`**: Finding minima/maxima of functions, root finding.
*   **`scipy.integrate`**: Numerical integration (calculating the area under a curve).
*   **`scipy.stats`**: A vast collection of statistical functions and distributions.
*   **`scipy.signal`**: Signal processing tools, filtering.
*   **`scipy.linalg`**: Advanced linear algebra routines.

You typically import the specific submodule you need, e.g., `from scipy.optimize import minimize`.

#### `Scikit-learn` (Machine Learning)
**Scikit-learn** is the premier library for classical machine learning in Python. It builds directly on NumPy and SciPy. Its key strengths are:

*   **Unified API:** All algorithms (called 'estimators') share a consistent interface:
    *   `estimator.fit(X, y)` to train the model.
    *   `estimator.predict(X_new)` to make predictions.
*   **Comprehensive:** It includes tools for classification, regression, clustering, dimensionality reduction, and more.
*   **Data Representation:** It expects data to be in NumPy arrays (or Pandas DataFrames). The features are a 2D array `X` and the target variable is a 1D array `y`.

**Example: Fitting a line**

```python
from sklearn.linear_model import LinearRegression

# Generate some noisy data: y = 2x + 1 + noise
X = np.array([0, 1, 2, 3, 4, 5]).reshape(-1, 1) # Features need to be 2D
y = np.array([1.1, 3.2, 4.9, 7.0, 9.1, 10.8])   # Target

# 1. Create a model instance
model = LinearRegression()

# 2. Train the model
model.fit(X, y)

# 3. Inspect the results
print(f"Learned slope: {model.coef_[0]:.2f}")
print(f"Learned intercept: {model.intercept_:.2f}")

# 4. Make a prediction
new_X = np.array([[6]])
prediction = model.predict(new_X)
print(f"Prediction for x=6: {prediction[0]:.2f}")
```