= is an assignment operator, it binds a name var to an object in memory, so the name var points to the object
Why can changing b change a?
if b=a, then b just is another label for a, so unless a is mutated using b, they will point to the same object 
Why does a list default argument remember old values? 
default arguments are evaluated once, when the function is defined. so, it is created one time and permanetely stores information. if you don't want it to be put into memory, use None to create it
When are is and == different? is looks at whether they point to the same object == looks at whether what is contained in the object is the same 

# 1. Two names, one object
a = [1, 2]
b = a
b.append(3)
print(a, a is b)

# 2. Equal but not identical
x = [1, 2]
y = [1, 2]
print(x == y, x is y)

# 3. The mutable default trap
def add(item, bucket=[]):
    bucket.append(item)
    return bucket
print(add(1))
print(add(2))

# 4. Shallow vs deep copy
import copy
outer = [[1], [2]]
shallow = copy.copy(outer)
deep = copy.deepcopy(outer)
outer[0].append(9)
print(shallow, deep)

# 5. What does a function get?
def change(lst):
    lst.append(1)
    lst = [99]
nums = [0]
change(nums)
print(nums)

# 6. The famous puzzle: predict carefully
t = (1, [2])
try:
    t[1] += [3]
except TypeError as e:
    print("error:", e)
print(t)