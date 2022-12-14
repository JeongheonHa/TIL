# asterisk(*)

π νμ΄μ¬μμμ asterisk(*)κΈ°νΈκ° κ°λ κΈ°λ₯μ λν΄μ μ΄ν΄λ³΄μ.

## β κ³±μκ³Ό κ±°λ­μ κ³±
μ€λͺ λμ  μ½λλ₯Ό λ³΄λ©΄ μ΄ν΄νκΈ° μ½λ€.
</br>

```python
print(2*3)  # 6, κ³±μ
print(2**3) # 8, κ±°λ­μ κ³±
```

## β λ¦¬μ€νΈμ νμ₯
*λ₯Ό μ¬μ©νμ¬ λ¦¬μ€νΈλ₯Ό κ°λ¨ν νμ₯ν  μ μλ€.
</br>

```python
list = [1]*5
print("list {}".format(list))   # list [1, 1, 1, 1, 1]
```
```python
tuple = (1,)*5
print("tuple {}".format(tuple)) # tuple (1, 1, 1, 1, 1)

# μ£Όμ
tuple = (1)*5
print("tuple {}".format(tuple)) # tuple 5
```

## β κ°λ³ μΈμ
`*arg`λ₯Ό λ§€κ°λ³μλ‘ μ§μ νλ©΄ ν¨μλ₯Ό μ μν  λ λ§€κ°λ³μλ‘ μ¬λ¬κ°μ μΈμλ₯Ό λ°μ μ¬ μ μλ κΈ°λ₯μ μ κ³΅νλ€.   

</br>

```python
def my_function(*kids):
  print("The youngest child is " + kids[2]) # The youngest child is Linus

my_function("Emil", "Tobias", "Linus") 
```
</br>

`**kwargs`λ₯Ό λ§€κ°λ³μλ‘ μ§μ νλ©΄ ν¨μλ₯Ό μ μν  λ λ§€κ°λ³μλ‘ λμλλ¦¬λ₯Ό λ°μ μ¬λ¬ μμμ μ κ·Όν  μ μλ κΈ°λ₯μ μ κ³΅νλ€.
</br>

```python
def my_function(**kid):
  print("His last name is " + kid["lname"]) # His last name is Refsnes

my_function(fname = "Tobias", lname = "Refsnes")
```

## β Unpacking
λ¦¬μ€νΈλ ννμ μΆλ ₯ν  λ `[1, 2, 3, 4, 5]`νμμΌλ‘ μΆλ ₯λλ κ²μ `1 2 3 4 5`νμμΌλ‘ μΆλ ₯μν¬ μ μλ κΈ°λ₯μ μ κ³΅νλ€.   

π μ΄λ κ² νλ©΄ forλ¬Έμ μ΄μ©ν΄μ λ¦¬μ€νΈμ λ΄μ©μ μΆλ ₯νμ§ μμλ λλ―λ‘ μ½λκ° ν¨μ¬ κ°κ²°ν΄μ§λ€.
</br>

```python
# list unpacking
test = [1, 2, 3, 4]
print(*test) # 1 2 3 4

# tuple unpacking
test = (5, 6, 7, 8)
print(*test) # 5 6 7 8
```

</br>
κ° λ³μ μ€ νλμ κ°λ³μ μΌλ‘ ν λΉνκ³  μΆμ κ²½μ° λ€μκ³Ό κ°μ΄ μ¬μ©ν  μ μλ€.
</br>

```python
test = [1, 2, 3, 4, 5]

*a, b = test
print(a, b) # [1, 2, 3, 4], 5

a, *b, c = test
print(a, b, c) # 1, [2, 3, 4], 5
```


# References
- <https://www.w3schools.com/python/gloss_python_function_arbitrary_arguments.asp>
- <https://hwiyong.tistory.com/193>
