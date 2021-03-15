# Advent of Code

This notebook contains my solutions for the 2020 version of [Advent of Code](https://adventofcode.com/2020).

![Test Notebook](https://github.com/jnslk/AoC2020/workflows/test%20notebook/badge.svg)

- [Day 1: Report Repair](#day-1:-report-repair)
- [Day 2: Password Philosophy](#day-2:-password-philosophy)
- [Day 3: Toboggan Trajectory](#day-3:-toboggan-trajectory)
- [Day 4: Passport Processing](#day-4:-passport-processing)
- [Day 5: Binary Boarding](#day-5:-binary-boarding)
- [Day 6: Custom Customs](#day-6:-custom-customs)
- [Day 7: Handy Haversacks](#day-7:-handy-haversacks)
- [Day 8: Handheld Halting](#day-8:-handheld-halting)
- [Day 9: Encoding Error](#day-9:-encoding-error)
- [Day 10: Adapter Array](#day-10:-adapter-array)
- [Day 11: Seating System](#day-11:-seating-system)
- [Day 12: Rain Risk](#day-12:-rain-risk)
- [Day 13: Shuttle Search](#day-13:-shuttle-search)
- [Day 14: Docking Data](#day-14:-docking-data)
- [Day 15: Rambunctious Recitation](#day-15:-rambunctious-recitation)
- [Day 16: Ticket Translation](#Day-16:-ticket-translation)
- [Day 17: Conway Cubes](#day-17:-conway-cubes)
- [Day 18: Operation Order](#day-18:-operation-order)
- [Day 19: Monster Messages](#day-19:-monster-messages)

### Imports and Dataimport


```python
from functools   import lru_cache
from itertools   import count, product
from typing      import Tuple, List, Dict
from collections import deque, Counter
from copy        import deepcopy
from math        import gcd, prod

import re
import ast
import operator
```


```python
def data(day: int, parser=str, sep='\n') -> list:
    "Split the day's input file into sections separated by `sep`, and apply `parser` function to each."
    with open(f'../data/day{day}.txt') as f:
        sections = f.read().rstrip().split(sep)
        return list(map(parser, sections))
```

# Day 1: Report Repair

## Part 1
The first challenge is to find two numbers in a list that add up to 2020 and return their product.


```python
test1_input = [1721, 979, 366, 299, 675, 1456]
# 1721 + 299 = 2020 and 1721 * 299 = 514579
test1_1output = 514579

def twosum(nums):
    complements = set()
    for x in nums:
        y = 2020 - x
        
        if x in complements:
            return x * y
        
        complements.add(y)
        
assert twosum(set(test1_input)) == test1_1output

input1 = set(data(1, int))
twosum(input1)
```




    926464



## Part 2
The second part of the puzzle is to find the three distinct numbers that add to 2020 and return their product.


```python
test1_input = [1721, 979, 366, 299, 675, 1456]
# 979 + 366 + 675 = 2020 and 979 * 366 * 675 = 241861950
test1_2output = 241861950

def threesum(nums):
    for x in nums:
        complements = set()
        yz = 2020 - x

        for y in nums:
            z = yz - y

            if y in complements:
                return x * y * z

            complements.add(z)

assert threesum(set(test1_input)) == test1_2output

input1 = set(data(1, int))
threesum(input1)
```




    65656536



# Day 2: Password Philosophy

## Part 1
The First part of this days challenge is to count how many passwords are valid according to their policies.

A given password string is considered valid if it contains between min and max instances of a specific character, where min, max and the character are specified in the policy.


```python
test2_input1 = '1-3 a: abcde'
test2_input2 = '1-3 b: cdefg'
test2_input3 = '2-9 c: ccccccccc'
test2_1output1 = True
test2_1output2 = False
test2_1output3 = True

Pw_Policy = Tuple[int, int, str, str]

def parse_pw_policy(line: str) -> Pw_Policy:
    "Given '1-3 b: cdefg', return (1, 3, 'b', 'cdefg')."
    mmin, mmax, letter, pw = re.findall(r'[^-:\s]+', line)
    return (int(mmin), int(mmax), letter, pw)

def check_pw(policy) -> bool:
    mmin, mmax, letter, pw = policy
    return mmin <= pw.count(letter) <= mmax

assert check_pw(parse_pw_policy(test2_input1)) == test2_1output1
assert check_pw(parse_pw_policy(test2_input2)) == test2_1output2
assert check_pw(parse_pw_policy(test2_input3)) == test2_1output3

input2: List[Tuple] = data(2, parse_pw_policy)

sum(map(check_pw, input2))
```




    378



## Part 2
The second part of this puzzle changes the interpretation of the password policy.
This digits that used to express min and max before now denote a position in the password string, however the index starts at 1.

A password is considered valid with this new policy if it has the specified letter at exactly one of the two given positions. How many passwords are valid?


```python
test2_2output1 = True
test2_2output2 = False
test2_2output3 = False

def check_pw2(policy, offset=1) -> bool:
    first, second, letter, pw = policy
    return (pw[first - offset] == letter) ^ (pw[second - offset] == letter)

assert check_pw2(parse_pw_policy(test2_input1)) == test2_2output1
assert check_pw2(parse_pw_policy(test2_input2)) == test2_2output2
assert check_pw2(parse_pw_policy(test2_input3)) == test2_2output3

sum(map(check_pw2, input2))
```




    280



# Day 3: Toboggan Trajectory

## Part 1
This challenge provides a map that contains '.' for free spaces and '#' for trees.

The questions is how many trees are encountered for a given map with a path that takes 3 steps right and 1 down until the bottom of the map is reached with the assumption that the pattern specified in the map repeats infinitely to the right.


```python
test3_input = ['..##.......',
               '#...#...#..',
               '.#....#..#.',
               '..#.#...#.#',
               '.#...##..#.',
               '..#.##.....',
               '.#.#.#....#',
               '.#........#',
               '#.##...#...',
               '#...##....#',
               '.#..#...#.#']
test3_1output = 7

Grid = List[str]

def count_trees_along_slope(grid, dx=3, dy=1, tree='#'):
    height = len(grid)
    width = len(grid[0])
    return sum((grid[row][col % width] == tree)
               for row, col in zip(range(0, height, dy), count(0, dx)))

assert count_trees_along_slope(test3_input, dx=3, dy=1, tree='#') == test3_1output

input3 : Grid = data(3)

count_trees_along_slope(input3)
```




    220



## Part 2
The second challenge for this day asks for the product of the encountered trees for paths with different slopes.


```python
test3_2output = 336

def count_trees_along_all_slopes(grid):
    def t(dx, dy):
        return count_trees_along_slope(grid, dx, dy)
    return t(1, 1) * t(3, 1) * t(5, 1) * t(7, 1) * t(1, 2)

assert count_trees_along_all_slopes(test3_input) == test3_2output

count_trees_along_all_slopes(input3)
```




    2138320800



# Day 4: Passport Processing

## Part 1
For this challenge a batch of passports is given. Each passport is represented as a sequence of key:value pairs. 
A passport is valid if it contains all expected fields while country id is optional. How many passports are valid?


```python
test4_input1 = "ecl:gry pid:860033327 eyr:2020 hcl:#fffffd byr:1937 iyr:2017 cid:147 hgt:183cm"
test4_input2 = "iyr:2013 ecl:amb cid:350 eyr:2023 pid:028048884 hcl:#cfa07d byr:1929"
test4_input3 = "hcl:#ae17e1 iyr:2013 eyr:2024 ecl:brn pid:760753108 byr:1931 hgt:179cm"
test4_input4 = "hcl:#cfa07d eyr:2025 pid:166559648 iyr:2011 ecl:brn hgt:59in"

test4_output1 = True
test4_output2 = False
test4_output3 = True
test4_output4 = False

Passport = dict

def parse_passport(passport_string: str) -> Passport:
    return Passport(re.findall(r'([a-z]+):([^\s]+)', passport_string))

required_fields = {'byr', 'iyr', 'eyr', 'hgt', 'hcl', 'ecl', 'pid'}

def check_passport_fields(Passport) -> bool:
    return required_fields.issubset(Passport.keys())

assert check_passport_fields(parse_passport(test4_input1)) == test4_output1
assert check_passport_fields(parse_passport(test4_input2)) == test4_output2
assert check_passport_fields(parse_passport(test4_input3)) == test4_output3
assert check_passport_fields(parse_passport(test4_input4)) == test4_output4

input4 : List[Passport] = data(4, parse_passport, '\n\n')

sum(map(check_passport_fields, input4))
```




    254



## Part 2
The second part of the puzzle requires passports to not only posses all required fields but also that the values of these fields obey specific rules. How many passports are valid according to the new stricter rules?


```python
test4_2input1 = "eyr:1972 cid:100 hcl:#18171d ecl:amb hgt:170 pid:186cm iyr:2018 byr:1926"
test4_2input2 = "iyr:2019 hcl:#602927 eyr:1967 hgt:170cm ecl:grn pid:012533040 byr:1946"
test4_2input3 = "hcl:dab227 iyr:2012 ecl:brn hgt:182cm pid:021572410 eyr:2020 byr:1992 cid:277"
test4_2input4 = "hgt:59cm ecl:zzz eyr:2038 hcl:74454a iyr:2023 pid:3556412378 byr:2007"

test4_2output1 = False
test4_2output2 = False
test4_2output3 = False
test4_2output4 = False

test4_2input5 = "pid:087499704 hgt:74in ecl:grn iyr:2012 eyr:2030 byr:1980 hcl:#623a2f"
test4_2input6 = "eyr:2029 ecl:blu cid:129 byr:1989 iyr:2014 pid:896056539 hcl:#a97842 hgt:165cm"
test4_2input7 = "hcl:#888785 hgt:164cm byr:2001 iyr:2015 cid:88 pid:545766238 ecl:hzl eyr:2022"
test4_2input8 = "iyr:2010 hgt:158cm hcl:#b6652a ecl:blu byr:1944 eyr:2021 pid:093154719"

test4_2output5 = True
test4_2output6 = True
test4_2output7 = True
test4_2output8 = True

def check_passport_values(Passport) -> bool:
    '''Passport fields are considered valid acording to these rules:
    byr (Birth Year) - four digits; at least 1920 and at most 2002.
    iyr (Issue Year) - four digits; at least 2010 and at most 2020.
    eyr (Expiration Year) - four digits; at least 2020 and at most 2030.
    hgt (Height) - a number followed by either cm or in:
        If cm, the number must be at least 150 and at most 193.
        If in, the number must be at least 59 and at most 76.
    hcl (Hair Color) - a # followed by exactly six characters 0-9 or a-f.
    ecl (Eye Color) - exactly one of: amb blu brn gry grn hzl oth.
    pid (Passport ID) - a nine-digit number, including leading zeroes.
    cid (Country ID) - ignored, missing or not.'''
    
    byr : bool = 1920 <= int(Passport['byr']) <= 2002
    iyr : bool = 2010 <= int(Passport['iyr']) <= 2020
    eyr : bool = 2020 <= int(Passport['eyr']) <= 2030
    hgt : bool = (((Passport['hgt'][-2:]=='cm') and (150 <= int(Passport['hgt'][:-2]) <= 193)) or
                  ((Passport['hgt'][-2:]=='in') and (59 <= int(Passport['hgt'][:-2]) <= 76)))
    hcl : bool = bool(re.match('#[0-9a-f]{6}$', Passport['hcl']))
    eyecolors = {'amb', 'blu', 'brn', 'gry', 'grn', 'hzl', 'oth'}
    ecl : bool = Passport['ecl'] in eyecolors
    pid : bool = bool(re.match('[0-9]{9}$', Passport['pid']))
    
    return all([byr, iyr, eyr, hgt, hcl, ecl, pid])

assert check_passport_values(parse_passport(test4_2input1)) == test4_2output1
assert check_passport_values(parse_passport(test4_2input2)) == test4_2output2
assert check_passport_values(parse_passport(test4_2input3)) == test4_2output3
assert check_passport_values(parse_passport(test4_2input4)) == test4_2output4
assert check_passport_values(parse_passport(test4_2input5)) == test4_2output5
assert check_passport_values(parse_passport(test4_2input6)) == test4_2output6
assert check_passport_values(parse_passport(test4_2input7)) == test4_2output7
assert check_passport_values(parse_passport(test4_2input8)) == test4_2output8

sum(check_passport_values(passport)
    for passport in input4 if check_passport_fields(passport))
```




    184



# Day 5: Binary Boarding

## Part 1
This puzzle provides a list of boarding passes that each specify one seat in the airplane. The airlane uses a binary space partioning approach for seating. Each boarding pass contains 10 characters, the first 7 are either 'F' or 'B' and specify the exact row. The last 3 characters can either be 'R' or 'L' and specify the column of the seat.

The airplane has 128 rows (0-127) and 8 columns(0-7). 'F' means front and indicates the lower half of the search space, 'B' means back and designates the upper half of the respective interval. For the column the convention is that 'L' can be interpreted as left and is meant to designate the lower half of the search space, while 'R' or right means the upper half.

Additionaly each seat in the airplane has a seat ID that is calculated by row * 8 + column.

The challenge is to find the highest seat ID of all provided boarding passes.


```python
test5_input1 = 'FBFBBFFRLR'
test5_input2 = 'BFFFBBFRRR'
test5_input3 = 'FFFBBBFRRR'
test5_input4 = 'BBFFBBFRLL'

test5_output1 = 357
test5_output2 = 567
test5_output3 = 119
test5_output4 = 820

Boarding_Pass = str

def parse_boarding_pass(Boarding_Pass) -> str:
    "translate the string to binary according to the specified conventions for rows and columns"
    table = str.maketrans('FBRL', '0110')
    return Boarding_Pass.translate(table)

def get_seat_ID(binary_boarding_pass) -> int:
    #the outcommented code was writen before realizing that
    #first 7 digits * 8 (equivalent to << 3) + last 3 digits is the same as original string in binary 😅
    #row = int(binary_boarding_pass[:-3], 2)
    #column = int(binary_boarding_pass[-3:], 2)
    #return row * 8 + column
    return int(binary_boarding_pass, 2)

assert get_seat_ID(parse_boarding_pass(test5_input1)) == test5_output1
assert get_seat_ID(parse_boarding_pass(test5_input2)) == test5_output2
assert get_seat_ID(parse_boarding_pass(test5_input3)) == test5_output3
assert get_seat_ID(parse_boarding_pass(test5_input4)) == test5_output4

input5 = data(5, parser=parse_boarding_pass)
max(map(get_seat_ID, input5))
```




    955



## Part 2
For the second part of this days puzzle a specific seat ID has to be found.
The wanted seat ID is not in the list of provided boarding passes.
This is not unambigously however so additionaly it is stated that the ID is not at the very front or back of the airplane but within the bulk of all the other seat IDs, as the adjacent IDs are also in the provided list.


```python
seat_IDs = set(map(get_seat_ID, input5))

def find_seat_ID():
    id = (set(range(min(seat_IDs), max(seat_IDs))) - (seat_IDs)).pop()
    return id

find_seat_ID()
```




    569



# Day 6: Custom Customs

## Part 1
This puzzle provides a list with answers of groups for customs declarations questions. Groups are separted by blank lines. A group can contain answers for multiple persons, all answers for one person are in a single line. There are 26 different custom questions and a positive answer is represented by the corresponding letter in the list. The challenge is to count all different questions that are group answered with yes and to sum up these counts for all groups.


```python
test6_input1 = '''abc

a
b
c

ab
ac

a
a
a
a

b'''
test6_output1 = 11

Group = List[str]

def parse_group(group_str: str) -> Group:
    return group_str.splitlines()

def count_groups(groups: List[Group]) -> int:
    return sum(len(set.union(*map(set, group))) for group in groups)

assert count_groups([parse_group(group) for group in test6_input1.split('\n\n')]) == test6_output1

input6: List[Group] = data(6, parse_group, sep='\n\n')
    
count_groups(input6)
```




    6596



## Part 2
For the second part the task is modified slightly to only count the answers in a group that were answered with yes by every person in the group and to then count these for all groups.


```python
test6_2output1 = 6

def count_groups2(groups: List[Group]) -> int:
    return sum(len(set.intersection(*map(set, group))) for group in groups)

assert count_groups2([parse_group(group) for group in test6_input1.split('\n\n')]) == test6_2output1

count_groups2(input6)
```




    3219



# Day 7: Handy Haversacks

## Part 1
This challenge provides a set of baggage rules. Bags are colorcoded and a rule specfies for a colored bag which other colored bags it can contain. The task is to find out how many other colored bags can contain a shiny golden bag either directly or within their other contained bags.


```python
test7_input1 = '''light red bags contain 1 bright white bag, 2 muted yellow bags.
dark orange bags contain 3 bright white bags, 4 muted yellow bags.
bright white bags contain 1 shiny gold bag.
muted yellow bags contain 2 shiny gold bags, 9 faded blue bags.
shiny gold bags contain 1 dark olive bag, 2 vibrant plum bags.
dark olive bags contain 3 faded blue bags, 4 dotted black bags.
vibrant plum bags contain 5 faded blue bags, 6 dotted black bags.
faded blue bags contain no other bags.
dotted black bags contain no other bags.'''

test7_output1 = 4

Bag = str
Rules = Dict[Bag, Dict[Bag, int]]

def parse_bag_rule(line: str) -> Tuple[Bag, Dict[Bag, int]]:
    line = re.sub(' bags?|[.]', '', line) 
    outer, inner = line.split(' contain ')
    return outer, dict(map(parse_inner, inner.split(', ')))

def parse_inner(text) -> Tuple[Bag, int]:
    "Return the color and number of inner bags."
    n, bag = text.split(maxsplit=1)
    return bag, (0 if n == 'no' else int(n))

assert parse_inner('1 bright white') == ('bright white', 1)
assert parse_inner('no other') == ('other', 0)
assert dict([parse_bag_rule(test7_input1.splitlines()[0])]) == {'light red': {'bright white': 1, 'muted yellow': 2}}
assert dict([parse_bag_rule(test7_input1.splitlines()[-1])]) == {'dotted black': {'other': 0}}

input7: Rules = dict(data(7, parse_bag_rule))

def count_bags_containing_at_least_one(rules, own_bag='shiny gold') -> int:
    @lru_cache(None)
    def contains(bag, own_bag) -> bool:
        contents = rules.get(bag, {})
        return (own_bag in contents or any(contains(inner, own_bag) for inner in contents))
    return sum(contains(bag, own_bag) for bag in rules)

assert count_bags_containing_at_least_one(dict(map(parse_bag_rule, test7_input1.splitlines()))) == test7_output1

count_bags_containing_at_least_one(input7)
```




    126



## Part 2
The second part of the challenge is to count all the bags that are contained in a shiny gold bag either directly or within the contained bags.


```python
test7_2output1 = 32

test7_input2 = '''shiny gold bags contain 2 dark red bags.
dark red bags contain 2 dark orange bags.
dark orange bags contain 2 dark yellow bags.
dark yellow bags contain 2 dark green bags.
dark green bags contain 2 dark blue bags.
dark blue bags contain 2 dark violet bags.
dark violet bags contain no other bags.'''

test7_output2 = 126

def count_bags_within(rules, own_bag='shiny gold') -> int:
    return sum(n + n * count_bags_within(rules, inner)
               for (inner, n) in rules[own_bag].items() if n > 0)

assert count_bags_within(dict(map(parse_bag_rule, test7_input1.splitlines()))) == test7_2output1
assert count_bags_within(dict(map(parse_bag_rule, test7_input2.splitlines()))) == test7_output2

count_bags_within(input7)
```




    220149



# Day 8: Handheld Halting

## Part1
This days puzzle provides a little assembly program, where each line consists of an opcode and an argument. Acc puts adds the argument to the accumulator, nop does no operation and is followe by the next instruction while jmp jumps to the relative address given by the argument. The challenge is to execute the given assembly program until it hits a loop for the first time and to return the accumulator value.


```python
test8_input1 = '''nop +0
acc +1
jmp +4
acc +3
jmp -3
acc -99
acc +1
jmp -4
acc +6'''

test8_output1 = 5

Instruction = Tuple[str, int]
Program = List[Instruction]

def parse_instruction(line: str) -> Instruction:
    opcode, argument = line.split(' ')
    return opcode, int(argument)

def accumulator_before_loop(program) -> int:
    program_counter = accumulator = 0
    already_executed= set()
    while True:
        if program_counter in already_executed:
            return accumulator
        already_executed.add(program_counter)
        opcode, argument = program[program_counter]
        if opcode == 'acc':
            accumulator += argument
            program_counter += 1
        if opcode == 'jmp':
            program_counter = program_counter + argument
        if opcode == 'nop':
            program_counter += 1

assert accumulator_before_loop([*map(parse_instruction, test8_input1.splitlines())]) == test8_output1
    
input8 = data(8, parse_instruction)

accumulator_before_loop(input8)
```




    1475



## Part 2
For part 2 exactly one jmp instruction can be swapped to nop or vice versa. The goal is to fix one instruction in the program, so that no loops occur. After the program terminates the accumulator has to be returned for this puzzle.


```python
test8_output2 = 8

def run(program) -> int:
    program_counter = accumulator = 0
    already_executed= set()
    while program_counter < len(program):
        if program_counter in already_executed:
            return 0
        already_executed.add(program_counter)
        opcode, argument = program[program_counter]
        if opcode == 'acc':
            accumulator += argument
            program_counter += 1
        if opcode == 'jmp':
            program_counter = program_counter + argument
        if opcode == 'nop':
            program_counter += 1
    return accumulator

def swap(instruction) -> Instruction:
    opcode, argument = instruction
    if opcode == 'jmp':
        opcode = 'nop'
    elif opcode == 'nop':
        opcode = 'jmp'
    return (opcode, argument)

assert swap(('nop', 0)) == ('jmp', 0)
assert swap(('acc', 1)) == ('acc', 1)
assert swap(('jmp', 4)) == ('nop', 4)

def fix_program(program) -> int:
    for index in range(len(program)):
        program[index] = swap(program[index])
        if run(program) != 0:
            return run(program)
        program[index] = swap(program[index])
    
assert fix_program([*map(parse_instruction, test8_input1.splitlines())]) == test8_output2

fix_program(input8)
```




    1270



# Day 9: Encoding Error

## Part 1
This puzzle provides a list of integers. After an preamble of 25 numbers every following number can be obtained as the sum of 2 of the previous 25 numbers. The challenge is to find the first number in the list, that violates this property.


```python
test9_input1 = '''35
20
15
25
47
40
62
55
65
95
102
117
150
182
127
219
299
277
309
576'''

test9_output1 = 127

def check_two_sum(numbers, target) -> bool:
    complements = set()
    for x in numbers:
        if x in complements:
            return True
        complements.add(target - x)
        
    return False

def find_outlier(numbers, windowsize=25) -> int:
    for i, target in enumerate(numbers[windowsize:]):
        if not check_two_sum(numbers[i:i + windowsize], target):
            return target

assert find_outlier([*map(int, test9_input1.splitlines())], 5) == test9_output1

input9 = data(9, int)

find_outlier(input9, 25)
```




    25918798



## Part 2
For the second part of the challenge the goal is to find a contiguous sequence of at least 2 numbers in the list that add up to the outlier from part 1. The sum of the minimum and maximum of this sequence has to be returned.


```python
test9_output2 = 62

def find_weakness(numbers, windowsize=25) -> int:
    outlier = find_outlier(numbers, windowsize)
    sequence = find_sequence(numbers, outlier)
    return min(sequence) + max(sequence)

def find_sequence(numbers, outlier):
    sequence = deque()
    sequence_sum = 0
    for num in numbers:
        if sequence_sum < outlier:
            sequence.append(num)
            sequence_sum += num
        if sequence_sum == outlier and len(sequence) >= 2:
            return sequence
        while sequence_sum > outlier:
            sequence_sum -= sequence.popleft()

assert find_weakness([*map(int, test9_input1.splitlines())], 5) == test9_output2

find_weakness(input9, 25)
```




    3340942



# Day 10: Adapter Array

## Part1
This puzzle provides a list of joltage values for charging adapter. Each adapter is rated for joltages 1, 2, or 3, jolts below its own joltage as input. The charging outlet in this puzzle has a joltage of 0 and the device that will be charged is rated for a joltage that is equal to that of the adapter with the largest joltage + 3.

The task is to construct a chain that uses all adapters and count the distribution off joltage difference between outlet, adapters and device. The number of all differences of 1 multiplied by the number of all differences of 3 has to be returned as solution.


```python
test10_input1 = '''16
10
15
5
1
11
7
19
6
12
4'''

test10_output1 = 35

test10_input2 = '''28
33
18
42
31
14
46
20
48
47
24
23
49
45
19
38
39
11
1
32
25
35
8
17
7
9
4
2
34
10
3'''

test10_output2 = 220

def build_adapter_chain(adapters) -> int:
    chain = [0] + sorted(adapters) + [max(adapters) + 3]
    diffs = Counter(chain[i + 1] - chain[i] for i in range(len(chain) - 1))
    return diffs[1] * diffs[3]

assert build_adapter_chain([*map(int, test10_input1.splitlines())]) == test10_output1
assert build_adapter_chain([*map(int, test10_input2.splitlines())]) == test10_output2

input10 = data(10, int)

build_adapter_chain(input10)
```




    2738



## Part 2
The Solution to the second part of this days challenge is the number of all valid combinations of adapters from outlet to device.


```python
test10_2output1 = 8
test10_2output2 = 19208

def count_arrangements(adapters) -> int:
    chain = [0] + sorted(adapters) + [max(adapters) + 3]
    return possible_solutions(tuple(chain), 0)

@lru_cache(None)
def possible_solutions(chain, previous):
    if previous == len(chain) - 1:
        return 1

    total = 0
    for i in range(previous + 1, min(previous + 4, len(chain))):
        if chain[i] - chain[previous] <= 3:
            total += possible_solutions(chain, i)

    return total

assert count_arrangements([*map(int, test10_input1.splitlines())]) == test10_2output1
assert count_arrangements([*map(int, test10_input2.splitlines())]) == test10_2output2

count_arrangements(input10)
```




    74049191673856



# Day 11: Seating System

## Part 1
Todays puzzle is a variation of Conways [Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life). The input represents a grid of a seating map. Each position can either be an empty seat (L), occupied (#) or a floor tile (.) and the following rules determine how a seat can change its state depending on its own state and on that of its 8 immidiate neighbour seats.

- If a seat is empty (L) and there are no occupied seats adjacent to it, the seat becomes occupied.
- If a seat is empty (L) and there are no occupied seats adjacent to it, the seat becomes occupied.
- If a seat is occupied (#) and four or more seats adjacent to it are also occupied, the seat becomes empty. Otherwise, the seat's state does not change.

The task is to simulate the given seat map until it stops changing and count the number of occupied seats.


```python
test11_input1 = '''L.LL.LL.LL
LLLLLLL.LL
L.L.L..L..
LLLL.LL.LL
L.LL.LL.LL
L.LLLLL.LL
..L.L.....
LLLLLLLLLL
L.LLLLLL.L
L.LLLLL.LL'''

test11_output1 = 37

Grid = List[str]

empty, occupied, floor = 'L#.'

def check_neighbors(grid, row, cell) -> int:
    deltas = ((-1, -1), (0, -1), (1, -1),
              (-1,  0),          (1,  0),
              (-1, +1), (0, +1), (1, +1)) 
    total = 0
    for dr, dc in deltas:
        nr, nc = row + dr, cell + dc
        if 0 <= nr < len(grid) and 0 <= nc < len(grid[0]):
            total += grid[nr][nc] == occupied
    return total


def run(grid, neighbor_function, crowded=4) -> int:
    while True:
        previous = deepcopy(grid)
        
        for r, row in enumerate(previous):
            for c, cell in enumerate(row):
                if cell == floor:
                    continue
                    
                N = neighbor_function(previous, r, c)
                
                if cell == empty and N == 0:
                    grid[r][c] = occupied
                elif cell == occupied and N >= crowded:
                    grid[r][c] = empty
                    
        if grid == previous:
            return sum(row.count(occupied) for row in grid)

        previous = grid

assert run(list(map(list, test11_input1.splitlines())), check_neighbors) == test11_output1

input11 = data(11, list)
run(deepcopy(input11), check_neighbors)
```




    2476



## Part 2
For the second Part there are two rule changes:
- An occupied seat turns empty if 5 or more (instead of 4) neigboring seats are also occupied.
- Instead of considering just the eight immediately adjacent seats, now the first seat visible across floor tiles in each of the eight directions is considered to determine how many neighboring seats are occupied.

The solution is again the number of occupied seats after the simulation converges.


```python
test11_2output1 = 26

def check_neighbors_visible(grid, row, cell) -> int:
    deltas = ((-1, -1), (0, -1), (1, -1),
              (-1,  0),          (1,  0),
              (-1, +1), (0, +1), (1, +1)) 
    total = 0
    for dr, dc in deltas:
        nr, nc = row + dr, cell + dc
        while 0 <= nr < len(grid) and 0 <= nc < len(grid[0]):
            if grid[nr][nc] != floor:
                total += grid[nr][nc] == occupied
                break
            nr += dr
            nc += dc
    return total

assert run(list(map(list, test11_input1.splitlines())), check_neighbors_visible, crowded=5) == test11_2output1

run(input11, check_neighbors_visible, crowded=5)
```




    2257



# Day 12: Rain Risk

## Part 1
This puzzle provides a list of navigation actions for a ship. Each action starts with a single letter followed by a value and has the following meaning:
- Action N means to move north by the given value.
- Action S means to move south by the given value.
- Action E means to move east by the given value.
- Action W means to move west by the given value.
- Action L means to turn left the given number of degrees.
- Action R means to turn right the given number of degrees.
- Action F means to move forward by the given value in the direction the ship is currently facing.

The ship starts facing east and only the two actions left and right can change the orientation of the ship.

What is the [Manhattan distance](https://en.wikipedia.org/wiki/Taxicab_geometry) between the starting position and the ships position after following all provided navigation instructions?


```python
test12_input1 = '''F10
N3
F7
R90
F11'''

test12_output1 = 25

Point = Course = Tuple[int, int]

north, east, south, west = 'NESW'
left, right, forward = 'LRF'

directions = {north: (0, 1), east: (1, 0), south: (0, -1), west: (-1, 0)}

def navigate(instructions, position=(0, 0), course=(directions[east])) -> int:
    for  command, value in instructions:
        if command == left or command == right:
            course = turn(course, command, value)
        elif command == forward:
            position = move(value, position, course)
        else:
            position = move(value, position, directions[command])
        
    return manhattan_distance(position[0], position[1])
        
def turn(course, command, value) -> Course:
    if value % 360 == 0:
        return course
    elif ((value % 360 == 90) and (command == right)) or ((value % 360 == 270) and (command == left)):
        return (course[1], course[0] * -1)
    elif value % 360 == 180:
        return tuple([-1 * coord for coord in course])
    else: # equivalent to value % 360 == 270) and command == right or value % 360 == 90 and command == left
        return (course[1] * -1, course[0])
        
def move(value, position, direction) -> Point:
    return (position[0] + value * direction[0], position[1] + value * direction[1])

def manhattan_distance(x, y) -> int:
    return abs(x) + abs(y)

def parse_nav_instruction(line) -> Instruction:
    return line[0], int(line[1:])

assert navigate([*map(parse_nav_instruction, test12_input1.splitlines())]) == test12_output1

input12 = data(12, parse_nav_instruction)
navigate(input12)
```




    1106



## Part 2
For the second part of the puzzle a waypoint is introduced and the rules change:
- Action N means to move the waypoint north by the given value.
- Action S means to move the waypoint south by the given value.
- Action E means to move the waypoint east by the given value.
- Action W means to move the waypoint west by the given value.
- Action L means to rotate the waypoint around the ship left (counter-clockwise) the given number of degrees.
- Action R means to rotate the waypoint around the ship right (clockwise) the given number of degrees.
- Action F means to move forward to the waypoint a number of times equal to the given value.

The waypoint starts 10 units east and 1 unit north relative to the ship. The waypoint is relative to the ship; that is, if the ship moves, the waypoint moves with it.

The solution is again the manhattan distance between the ships starting location and the final position after following the updatet navigation instructions.


```python
test12_output2 = 286

def navigate_waypoint(instructions, position=(0, 0), waypoint=(10, 1)) -> int:
    for command, value in instructions:
        if command == left or command == right:
            waypoint = turn(waypoint, command, value)
        elif command == forward:
            position = move(value, position, waypoint)
        else:
            waypoint = move(value, waypoint, directions[command])
        
    return manhattan_distance(position[0], position[1])

assert navigate_waypoint([*map(parse_nav_instruction, test12_input1.splitlines())]) == test12_output2

navigate_waypoint(input12)
```




    107281



# Day 13: Shuttle Search

## Part 1
This challenge deals with a bus-shuttle-schedule. The schedule is defined based on a timestamp that measures the number of minutes since some fixed reference point in the past.

At timestamp 0 all buses start their routes from the port and drive on to eventually come back to the port where they started and then repeat their tour infinitely. However the duration of the different bus tours differs and the duration of each bus' tour is represented in its ID which gives the duration in minutes.

So the bus with ID 15 would depart every 15 minutes and so on...

The input to the puzzle provides an start timestamp and a schedule of busses with their IDs.
The solution is to find the bus ID of the earliest departing bus after the start timestamp multiplied by the waiting duration in minutes.


```python
test13_input1 = '''939
7,13,x,x,59,x,31,19'''

test13_output1 = 295

def parse_timetable(line):
    return [int(x) for x in line.split(',') if x != 'x']

def find_earliest_bus(timetable) -> int:
    departure = timetable[0][0]
    bus_id = min(timetable[1], key=lambda bus: bus - (departure % bus))        
    return bus_id * (bus_id - (departure % bus_id))

assert find_earliest_bus([*map(parse_timetable, test13_input1.splitlines())]) == test13_output1

input13_1 = data(13, parse_timetable)
find_earliest_bus(input13_1)
```




    296



## Part 2
The second part of the challenge is to find the earliest timestamp such that the first bus ID departs at that time and each subsequent listed bus ID departs at that subsequent minute, that is represented by the index in the schedule.

So the first bus(index 0) in the schedule starts at timestamp t, the second bus(index 1) in the schedule starts at timestamp t+1, the nth bus starts at t+n and so on...


```python
test13_2output1 = 1068781

test13_input2 = '17,x,13,19'
test13_input3 = '67,7,59,61'
test13_input4 = '67,x,7,59,61'
test13_input5 = '67,7,x,59,61'
test13_input6 = '1789,37,47,1889'

test13_output2 = 3417
test13_output3 = 754018
test13_output4 = 779210
test13_output5 = 1261476
test13_output6 = 1202161486

def find_earliest_timestamp(timetable) -> int:
    bus_periods = []
    for index, value in enumerate(timetable):
        if value != 'x':
            bus_periods.append((index, int(value)))
    time, step = bus_periods[0]
    for delta, period in bus_periods[1:]:
        for time in count(time, step):
            if (time + delta) % period == 0:
                break
        step = least_common_multiple(step, period)
    return time

def least_common_multiple(x, y) -> int:
    return x * y // gcd(x, y)

assert find_earliest_timestamp(test13_input1.splitlines()[1].split(',')) == test13_2output1
assert find_earliest_timestamp(test13_input2.split(',')) == test13_output2
assert find_earliest_timestamp(test13_input3.split(',')) == test13_output3
assert find_earliest_timestamp(test13_input4.split(',')) == test13_output4
assert find_earliest_timestamp(test13_input5.split(',')) == test13_output5
assert find_earliest_timestamp(test13_input6.split(',')) == test13_output6

input13_2 = data(13)
find_earliest_timestamp(input13_2[1].split(','))
```




    535296695251210



# Day 14: Docking Data

## Part 1
This puzzle provides an initialization program. The instructions can do one of two things.
- update the bitmask
- write a value to a memory address

Values and memory addresses are both 36-bit unsigned integers. The current bitmask is applied to values immediately before they are written to memory: a 0 or 1 overwrites the corresponding bit in the value, while an X leaves the bit in the value unchanged.

All memory values are initialized with 0. What is the sum of all values in memory after the program finishes?


```python
test14_input1 = '''mask = XXXXXXXXXXXXXXXXXXXXXXXXXXXXX1XXXX0X
mem[8] = 11
mem[7] = 101
mem[8] = 0'''

test14_output1 = 165

memline = re.compile(r'mem\[(\d+)\] = (\d+)')

def parse_program(line) -> tuple:
    if line.startswith('mask'):
        return ('mask', line.split(' = ')[1])
    else:
        return tuple(map(int, memline.findall(line)[0]))

def binary_36bit(i) -> str:
    return f'{i:036b}'

def masked(mask, bitstring) -> int:
    return int(''.join((m if m != 'X' else b for m, b in zip(mask, bitstring))), 2)

def run_program(program) -> int:
    mem = dict()
    mask = binary_36bit(0)
    for address, value in program:
        if address == 'mask':
            mask = value
        else:
            mem[address] = masked(mask, binary_36bit(value))
            
    return sum(mem[key] for key in mem)

assert run_program([*map(parse_program, test14_input1.splitlines())]) == test14_output1

input14 = data(14, parse_program)
run_program(input14)
```




    5055782549997



## Part 2
Rules changed slightly for the second part. Values writen to memory are not affected anymore by the mask, instead the mask is apllied to the address:
- If the bitmask bit is 0, the corresponding memory address bit is unchanged.
- If the bitmask bit is 1, the corresponding memory address bit is overwritten with 1.
- If the bitmask bit is X, the corresponding memory address bit is floating.

Floating means the address bit takes all possible states, so the write hits not one single memory address but potentially multiple for all possible combinations of 0 and 1 for the floating bits in the address. What is the sum of all values in memory after the program finishes?


```python
test14_input2 = '''mask = 000000000000000000000000000000X1001X
mem[42] = 100
mask = 00000000000000000000000000000000X0XX
mem[26] = 1'''

test14_output2 = 208

def masked2(mask, bitstring) -> int:
    return ''.join(('X' if m == 'X' else m if m == '1' else b for m, b in zip(mask, bitstring)))

def run_program2(program) -> int():
    mem = dict()
    mask = binary_36bit(0)
    for address, value in program:
        if address == 'mask':
            mask = value
        else:
            address = masked2(mask, binary_36bit(address))
            if address.count('X') == 0:
                mem[int(address, 2)] = value
            else:
                addresses = []
                for m, a in zip(mask, address):
                    if m == 'X':
                        addresses.append('10')
                    elif m == '1':
                        addresses.append('1')
                    else:
                        addresses.append(a)
                        
                for addr in product(*addresses):
                    mem[int(''.join(addr), 2)] = value
        
    return sum(mem[key] for key in mem)

assert run_program2([*map(parse_program, test14_input2.splitlines())]) == test14_output2

run_program2(input14)
```




    4795970362286



# Day 15: Rambunctious Recitation

## Part 1
This days puzzle is about a memory game. In this game, the players take turns saying numbers. They begin by taking turns reading from a list of starting numbers. Then, each turn consists of considering the most recently spoken number:

- If that was the first time the number has been spoken, the current player says 0.
- Otherwise, the number had been spoken before; the current player announces how many turns apart the number is from when it was previously spoken.

So, after the starting numbers, each turn results in either 0 (if the last number is new) or an age (if the last number is a repeat).

What will be the 2020th number spoken?


```python
test15_input1 = [0,3,6]

test15_output1 = 436

def memory_game(numbers, end=2020) -> int:
    last_index = {num: index for index, num in enumerate(numbers)}
    last = 0
    
    for index in range(len(numbers), end - 1):
        if last in last_index:
            new = index - last_index[last]
        else:
            new = 0

        last_index[last] = index
        last = new

    return new

assert memory_game(test15_input1) == test15_output1

input15 = data(15, int)

memory_game(input15)
```




    870



## Part 2 
For the second part not the number after turn 2020 but instead 30000000 is wanted.


```python
test15_2output1 = 175594

assert memory_game(test15_input1, 30000000) == test15_2output1

memory_game(input15, 30000000)
```




    9136



# Day 16: Ticket Translation

## Part 1
Todays puzzle is about deciphering the information of a train ticket. The text on the ticket is in a unknown language, so the input of the train ticket contains only numbers in certain fields. Additionaly the numbers of nearby tickets are available and a table with rules for the tidket fields that list the name of fields and ranges for valid numbers in those fields. 

The challenge for the first part is to find all of the nearby tickets that contain values that aren´t valid for any field. What is the sum of all those invalid numbers across all tickets?


```python
test16_input1 = '''class: 1-3 or 5-7
row: 6-11 or 33-44
seat: 13-40 or 45-50

your ticket:
7,1,14

nearby tickets:
7,3,47
40,4,50
55,2,20
38,6,12'''

test16_output1 = 71

field_re = re.compile(r'(\d+)-(\d+) or (\d+)-(\d+)')
def parse_field_ranges(line):
    a, b, c, d = field_re.findall(line)[0]
    return set(range(int(a), int(b) + 1)).union(set(range(int(c), int(d) + 1)))

def parse_ticket(line):
    return [*map(int, line.split(','))]

def invalid_tickets(reference_doc):
    all_fields = set().union(*[parse_field_ranges(line) for line in reference_doc[0]])
    tickets = [parse_ticket(line) for line in reference_doc[2][1:]]
    return sum(value for ticket in tickets for value in ticket
               if value not in all_fields)
    
assert invalid_tickets([*map(str.splitlines, test16_input1.split('\n\n'))]) == test16_output1
    
input16 = data(16, str.splitlines, sep='\n\n')

invalid_tickets(input16)
```




    25895



## Part 2
The second part of the challenge is to identify which numbers on the train ticket correspond to which field. All invalid tickets have to be discarded and the numbers on the nearby tickets together with the given train ticket can be used together with the rules for field values. What is the product of all values on the train ticket for fields that start with 'departure' ?


```python
test16_input2 = '''class: 0-1 or 4-19
row: 0-5 or 8-19
seat: 0-13 or 16-19

your ticket:
11,12,13

nearby tickets:
3,9,18
15,1,5
5,14,9
'''

test16_output2 = {'class': 12, 'row': 11, 'seat': 13}

def parse_field(line) -> Tuple[str, set]:
    field = line.split(':')[0]
    a, b, c, d = field_re.findall(line)[0]
    return field, set(range(int(a), int(b) + 1)).union(set(range(int(c), int(d) + 1)))

def infer_ticket(reference_doc) -> dict:
    fields = dict(map(parse_field, reference_doc[0]))
    all_fields = set().union(*fields.values())
    own_ticket =  parse_ticket(reference_doc[1][1])
    tickets = [parse_ticket(line) for line in reference_doc[2][1:]]
    valid_tickets = [t for t in tickets + [own_ticket] if check_ticket(t, all_fields)]
    possible_fields = [set(fields) for _ in range(len(own_ticket))]
    while any(len(p) > 1 for p in possible_fields):
        for field_name, i in invalid_fields(valid_tickets, fields):
            possible_fields[i] -= {field_name}
            if len(possible_fields[i]) == 1:
                eliminate_others(possible_fields, i)
    order = {field: i for i, [field] in enumerate(possible_fields)}
    return {field: own_ticket[order[field]] for field in order}

def check_ticket(ticket, fields) -> bool:
    return all(value in fields for value in ticket)

def invalid_fields(tickets, fields) -> Tuple[str, int]:
    return ((field_name, i) for ticket in tickets for i in range(len(ticket))
            for field_name in fields 
            if ticket[i] not in fields[field_name])

def eliminate_others(possible_fields, i):
    for other in range(len(possible_fields)):
        if other != i:
            possible_fields[other] -= possible_fields[i]

def departure_product(ticket) -> int:
    return prod([ticket[field] for field in ticket if field.startswith('departure')])

assert infer_ticket([*map(str.splitlines, test16_input2.split('\n\n'))]) == test16_output2

departure_product(infer_ticket(input16))
```




    5865723727753



# Day 17: Conway Cubes

## Part 1
This days puzzle is Conway's Game of Life in three dimensions. It is played on a infinite 3-dimensional grid. At each integer coordinate of the grid given by (x, y, z) there is one cube that is either active or inactive.

At the start of the simulation, almost all cubes are inactive except for a pattern on a 2-dimensional slice at the bottom that is provided as input. In this pattern '#' denotes an active cube and '.' represents the inactive state.

The simulation then proceeds for 6 cycles with the following rules:

- Each cube only considers its 26 neighbouring cubes.
- Within a cycle all cubes change their states simultaneously.
- If a cube is active and exactly 2 or 3 of its neighbors are also active, the cube remains active. Otherwise, the cube becomes inactive.
- If a cube is inactive but exactly 3 of its neighbors are active, the cube becomes active. Otherwise, the cube remains inactive.

After 6 cycles, how many cubes are active?


```python
test17_input1 = '''.#.
..#
###'''

test17_output1 = 112

active = '#'

def simulate_cycles(grid, n=6) -> int:
    alive = set((x, y, 0) for x, y in product(range(len(grid)), range(len(grid[0]))) if grid[x][y] == active)
    
    for _ in range(n):
        alive = next_generation(alive)
    return len(alive)

def neighbors(x, y, z):
    yield from product(range(x - 1, x + 2), range(y - 1, y + 2), range(z - 1, z + 2))

def alive_neighbors(alive, coords) -> int:
    total = sum(neighbor in alive for neighbor in neighbors(*coords))
    # prevent own coordinates from being counted as neighbor
    total -= coords in alive
    return total

def all_neighbors(alive):
    return set(n for cube in alive for n in neighbors(*cube))

def next_generation(alive):
    new = set()
    for cube in all_neighbors(alive):
        N = alive_neighbors(alive, cube)
        if N == 3 or (N == 2 and cube in alive):
            new.add(cube)
    return new

assert simulate_cycles(test17_input1.splitlines()) == test17_output1

input17 = data(17)

simulate_cycles(input17)
```




    372



## Part 2
The second challenge is to solve the exact same problem but in 4 dimensions this time.


```python
test17_2output1 = 848

def simulate_cycles(grid, n=6, d=4) -> int:
    alive = set((x, y, *(d - 2) * (0, ))
                for x, y in product(range(len(grid)), range(len(grid[0])))
                if grid[x][y] == active)
    
    for _ in range(n):
        alive = next_generation(alive)
    return len(alive)

def neighbors(coords):
    ranges = ((c - 1, c, c + 1) for c in coords)
    yield from product(*ranges)

def alive_neighbors(alive, coords) -> int:
    total = sum(neighbor in alive for neighbor in neighbors(coords))
    # prevent own coordinates from being counted as neighbor
    total -= coords in alive
    return total

def all_neighbors(alive):
    return set(n for hypercube in alive for n in neighbors(hypercube))

assert simulate_cycles(test17_input1.splitlines(), d=3) == test17_output1
assert simulate_cycles(test17_input1.splitlines(), d=4) == test17_2output1

simulate_cycles(input17, d=4)
```




    1896



# Day 18: Operation Order

## Part 1
This puzzle consists of a list of math expressions that are a combination of natural numbers and addition and multiplication operators aswell as parentheses. However the conventional operator precedence of multiplication before addition is discarded for this challenge, only parentheses can influence which operations get evaluated first. 

Given this rule change to conventional math, what is the sum of all expressions in the list if evaluated?


```python
test18_input1 = '1 + 2 * 3 + 4 * 5 + 6'
test18_input2 = '1 + (2 * 3) + (4 * (5 + 6))'
test18_input3 = '2 * 3 + (4 * 5)'
test18_input4 = '5 + (8 * 3 + 9 + 3 * 4 * 3)'
test18_input5 = '5 * 9 * (7 * 3 * 3 + 9 * 3 + (8 + 6 * 4))'
test18_input6 = '((2 + 4 * 9) * (6 + 9 * 8 + 6) + 6) + 2 + 4 * 2'

test18_output1 = 71
test18_output2 = 51
test18_output3 = 26
test18_output4 = 437
test18_output5 = 12240
test18_output6 = 13632

operators = {'+': operator.add, '*': operator.mul}

def parse_expression(line) -> tuple:
    return ast.literal_eval(re.sub('([+*])', r",'\1',", line))

def evaluate_l2r(expr) -> int:
    if isinstance(expr, int):
        return expr
    else:
        a, op, b, *rest = expr
        x = operators[op](evaluate_l2r(a), evaluate_l2r(b))
        return x if not rest else evaluate_l2r((x, *rest))

assert evaluate_l2r(parse_expression(test18_input1)) == test18_output1
assert evaluate_l2r(parse_expression(test18_input2)) == test18_output2
assert evaluate_l2r(parse_expression(test18_input3)) == test18_output3
assert evaluate_l2r(parse_expression(test18_input4)) == test18_output4
assert evaluate_l2r(parse_expression(test18_input5)) == test18_output5
assert evaluate_l2r(parse_expression(test18_input6)) == test18_output6

input18 = data(18, parse_expression)

sum(map(evaluate_l2r, input18))
```




    14208061823964



## Part 2

The second part hast the twist that now addition has a higher precedence than multiplication. Again what is the sum of all expressions given this new rule?


```python
test18_2output1 = 231
test18_2output2 = 51
test18_2output3 = 46
test18_2output4 = 1445
test18_2output5 = 669060
test18_2output6 = 23340

def evaluate_add_first(expr) -> int:
    if isinstance(expr, int):
        return expr
    elif '*' in expr:
        i = expr.index('*')
        return evaluate_add_first(expr[:i]) * evaluate_add_first(expr[i + 1:])
    else:
        return sum(evaluate_add_first(x) for x in expr if x != '+')

assert evaluate_add_first(parse_expression(test18_input1)) == test18_2output1
assert evaluate_add_first(parse_expression(test18_input2)) == test18_2output2
assert evaluate_add_first(parse_expression(test18_input3)) == test18_2output3
assert evaluate_add_first(parse_expression(test18_input4)) == test18_2output4
assert evaluate_add_first(parse_expression(test18_input5)) == test18_2output5
assert evaluate_add_first(parse_expression(test18_input6)) == test18_2output6

sum(map(evaluate_add_first, input18))
```




    320536571743074



# Day 19: Monster Messages

## Part 1
This puzzle provides a set of rules and a set of strings. Each string is made up of a sequence containing either 'a' or 'b'. Rules start with their ID and can either be 'a', 'b' or a reference to other rule IDs. There is also a pipe operator to indicate alternative sets or rule IDs. There are no cycles in the references of rule IDs. A rule is matched by a string if it satisfies the pattern specified by the rule and there are no characters left to match after the rule ends. Given the set of rules, how many strings are matched by the first rule?


```python
test19_input1 = '''0: 4 1 5
1: 2 3 | 3 2
2: 4 4 | 5 5
3: 4 5 | 5 4
4: "a"
5: "b"

ababbb
bababa
abbbab
aaabbb
aaaabbb'''

test19_output1 = 2

def parse_message_rules(rules_list) -> dict:
    rules = {}
    
    for rule_str in rules_list:
        rule_ID, body = rule_str.split(': ')
        rule_ID = int(rule_ID)
        if '"' in body:
            rule = body[1]
        else:
            rule = []
            for option in body.split('|'):
                rule.append(tuple(map(int, option.split())))
        rules[rule_ID] = rule
    return rules

def count_valid_messages(rules, messages) -> int:
    rules = parse_message_rules(rules)
    rule_regex = re.compile('^' + construct_rule_regex(rules) + '$')
    return sum(bool(rule_regex.match(msg)) for msg in messages)

def construct_rule_regex(rules, rule=0) -> str:
    rule = rules[rule]
    if type(rule) is str:
        return rule
    options = []
    for option in rule:
        option = ''.join(construct_rule_regex(rules, r) for r in option)
        options.append(option)
    return '(' + '|'.join(options) + ')'

assert count_valid_messages(*map(str.splitlines, test19_input1.split('\n\n'))) == test19_output1

input19 = data(19, str.splitlines, sep='\n\n')

count_valid_messages(*input19)
```




    142



## Part 2

The second part of the challenge introduces two cycles in the rules 8 and 11. How many strings are matched with these changes by the first rule?


```python
test19_input2 = '''42: 9 14 | 10 1
9: 14 27 | 1 26
10: 23 14 | 28 1
1: "a"
11: 42 31
5: 1 14 | 15 1
19: 14 1 | 14 14
12: 24 14 | 19 1
16: 15 1 | 14 14
31: 14 17 | 1 13
6: 14 14 | 1 14
2: 1 24 | 14 4
0: 8 11
13: 14 3 | 1 12
15: 1 | 14
17: 14 2 | 1 7
23: 25 1 | 22 14
28: 16 1
4: 1 1
20: 14 14 | 1 15
3: 5 14 | 16 1
27: 1 6 | 14 18
14: "b"
21: 14 1 | 1 14
25: 1 1 | 1 14
22: 14 14
8: 42
26: 14 22 | 1 20
18: 15 15
7: 14 5 | 1 21
24: 14 1

abbbbbabbbaaaababbaabbbbabababbbabbbbbbabaaaa
bbabbbbaabaabba
babbbbaabbbbbabbbbbbaabaaabaaa
aaabbbbbbaaaabaababaabababbabaaabbababababaaa
bbbbbbbaaaabbbbaaabbabaaa
bbbababbbbaaaaaaaabbababaaababaabab
ababaaaaaabaaab
ababaaaaabbbaba
baabbaaaabbaaaababbaababb
abbbbabbbbaaaababbbbbbaaaababb
aaaaabbaabaaaaababaa
aaaabbaaaabbaaa
aaaabbaabbaaaaaaabbbabbbaaabbaabaaa
babaaabbbaaabaababbaabababaaab
aabbbbbaabbbaaaaaabbbbbababaaaaabbaaabba'''

test19_output2 = 12
test19_1output2 = 3

assert count_valid_messages(*map(str.splitlines, test19_input2.split('\n\n'))) == test19_1output2

def maybe(n):
    return (([], [n]))

def maybe_int(text: str):
    try:
        return int(text)
    except ValueError:
        return text

def parse_rule(line):
    ruleID, *body = line.split(': ')
    ruleID = int(ruleID)
    if '"' in body[0]:
        return (ruleID, [body[0][1]])
    body = [maybe_int(i) for i in body[0].split()]
    if '|' in body:
        i = body.index('|')
        body = [tuple((body[:i], body[i + 1:]))]
    return (ruleID, body)
    
def count_valid_messages2(rules, messages) -> int:
    rules = dict(map(parse_rule, rules))
    rules[8] = [42, maybe(8)]
    rules[11] = [42, maybe(11), 31]
    return sum([match(rules[0], msg, rules) == '' for msg in messages])

def match(pat, msg, rules):
    if pat and not msg:              # Failed to match whole pat
        return None 
    elif not pat:                    # Matched whole pat
        return msg 
    elif pat[0] == msg[0]:           # Matched first char; continue
        return match(pat[1:], msg[1:], rules)
    elif isinstance(pat[0], int):    # Look up the rule number 
        return match(rules[pat[0]] + pat[1:], msg, rules)
    elif isinstance(pat[0], tuple): # Match any of the choices
        for choice in pat[0]:
            m = match(choice + pat[1:], msg, rules)
            if m is not None:
                return m
    return None

assert count_valid_messages2(*map(str.splitlines, test19_input2.split('\n\n'))) == test19_output2

count_valid_messages2(*input19)
```




    294


