# set λ©μλ 

π `λ©μλ -> νμ`μ λ°ν νμμ μ μ΄ λμ κ²μλλ€.

## β set.add(elmnt) -> None
setμ μμλ₯Ό μΆκ°νλ€.   
λ§μ½ μμκ° μ‘΄μ¬ν  κ²½μ° μμλ μΆκ°λμ§ μλλ€.
</br>

### Parameter Values
elmnt : Required. The element to add to the set
</br>

### Example

```python
thisset = {"apple", "banana", "cherry"}
thisset.add("apple")
print(thisset)  # {'banana', 'apple', 'cherry'}
```
</br>

## β set.pop() -> type of elem
setμμ λλ€μΌλ‘ μμ νλλ₯Ό μ κ±°ν ν λ°ννλ€.
</br>

### Parameter Values
No parameter values.
</br>

### Example

```python
fruits = {"apple", "banana", "cherry"}
x = fruits.pop() 
print(x)    # cherry
```
</br>

## β set.remove(item) -> None
setμμ νΉμ  μμλ‘€ μ κ±°νλ€. λ§μ½ μμκ° μ‘΄μ¬νμ§ μμ κ²½μ° errorλ°μ

π discard()μ κΈ°λ₯μ λμΌνμ§λ§ μμκ° μ‘΄μ¬νμ§ μμΌλ©΄ errorκ° λ°μνλ€.
</br>

### Parameter Values
item : Required. The item to search for, and remove
</br>

### Example

```python
fruits = {"apple", "banana", "cherry"}
fruits.remove("banana") 
print(fruits)   # {'apple', 'cherry'}
```
</br>

## β set.discard(value) -> None
setμμ νΉμ  μμλ₯Ό μ κ±°νλ€. λ§μ½ μμκ° μ‘΄μ¬νμ§ μμλ errorκ° λ°μνμ§ μλλ€.

π remove()μ κΈ°λ₯μ λμΌνμ§λ§ μμκ° μ‘΄μ¬νμ§ μμλ errorκ° λ°μνμ§ μλλ€.
</br>

### Parameter Values
value : Required. The item to search for, and remove
</br>

### Example

```python
fruits = {"apple", "banana", "cherry"}
fruits.discard("banana") 
print(fruits)   # {'cherry', 'apple'}
```
</br>

## β set.clear() -> None
setμ λͺ¨λ  μμλ₯Ό μ κ±°νλ€.
</br>

### Parameter Values
No Parameters
</br>

### Example

```python
fruits = {"apple", "banana", "cherry"}
fruits.clear()
print(fruits)   # set()
```

## β set.copy()
setμ λ³΅μ¬λ³Έμ λ°ννλ€.
</br>

### Parameter Values
No parameters
</br>

### Example

```python
fruits = {"apple", "banana", "cherry"}
x = fruits.copy()
print(x)    # {'banana', 'apple', 'cherry'}
```
</br>

## β set.update(set) -> None
λ€λ₯Έ setλλ λ€λ₯Έ iterableμ μμλ€μ μΆκ°ν¨μΌλ‘μ¨ νμ¬ setμ μλ°μ΄νΈνλ€.

π λ§μ½ λ setμ μμκ° μ‘΄μ¬ν  κ²½μ° μλ°μ΄νΈλ setμλ μ€μ§ ν κ°λ§ μ‘΄μ¬νλ€.
</br>

### Parameter Values
set	: Required. The iterable insert into the current set
</br>

### Example

```python
x = {"apple", "banana", "cherry"}
y = {"google", "microsoft", "apple"}
x.update(y) 
print(x)    # {'microsoft', 'apple', 'google', 'banana', 'cherry'}
```
</br>

## β set.union(set1, set2...) -> set
μ¬λ¬ setμ ν©μ§ν©μ μλ―Ένλ€.
</br>

### Parameter Values
set1 : Required. The iterable to unify with
set2 : Optional. The other iterable to unify with. You can compare as many iterables as you like. (Separate each iterable with a comma)
</br>

### Example

```python
x = {"a", "b", "c"}
y = {"f", "d", "a"}
z = {"c", "d", "e"}
result = x.union(y, z)  # result = x | y | zμ κ°λ€.
print(result)   # {'c', 'd', 'e', 'a', 'f', 'b'}
```
</br>

## β set.intersection(set1, set2 ... etc) -> set
μ¬λ¬ setμ κ΅μ§ν©μ μλ―Ένλ€.
</br>

### Parameter Values
set1 : Required. The set to search for equal items in
set2 : Optional. The other set to search for equal items in. You can compare as many sets you like. (Separate the sets with a comma)
</br>

### Example

```python
x = {"a", "b", "c"}
y = {"c", "d", "e"}
z = {"f", "g", "c"}
result = x.intersection(y, z)   # result = x & y & zμ κ°λ€.
print(result)   # {'c'}
```
</br>

## β set.difference(set) -> set
λ λ²μ§Έ setμ μ‘΄μ¬νμ§ μκ³  μ²« λ²μ§Έ setμλ§ μ‘΄μ¬νλ μμλ€μ λ°ννλ€. (μ°¨μ§ν©μ μλ―Ένλ€.)
</br>

### Parameter Values
set	: Required. The set to check for differences in
</br>

### Example

```python
x = {"apple", "banana", "cherry"}
y = {"google", "microsoft", "apple"}
z = x.difference(y) # z = x - yμ κ°λ€.
print(z)    # {'banana', 'cherry'}
z = y.difference(x) # z = y - xμ κ°λ€. 
print(z)    # {'microsoft', 'google'}
```
</br>

## References
<https://www.w3schools.com/python/python_ref_set.asp>
