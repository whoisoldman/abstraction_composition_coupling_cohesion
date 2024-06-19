# abstraction_composition_coupling_cohesion
Ways to improve code quality.

+++START_EN+++

## TASK:

**Review the provided code example and identify issues such as low level of abstraction, high coupling, low cohesion, and layout problems (describe which classes and functions you found each issue in). Propose ways to improve the code quality in the example. Implement an improved version and explain how you addressed the identified issues.**

### Code Example:

```python
import math
import random

class Account:
  def __init__(self, bank, account_number: int, customer_name: str, customer_address: str, balance: float):
    self.account_number = account_number
    self.balance = balance
    self.customer_name = customer_name
    self.customer_address = customer_address
    self.bank = bank

  def deposit(self, amount: float):
    self.bank.accounts[self.account_number].balance += amount

  def withdraw(self, amount: float):
    if amount <= self.bank.accounts[self.account_number].balance:
      self.bank.accounts[self.account_number].balance -= amount
    else:
      raise ValueError("Insufficient funds")


class Bank:
  def __init__(self):
    self.accounts = {}

  def create_account(self, account_number: int, customer_name: str, customer_address: str, initial_balance: float) -> Account:
    account = Account(self, account_number, customer_name, customer_address, initial_balance)
    self.accounts[account_number] = account
    return account

  def get_account(self, account_number: int) -> Account:
    if account_number in self.accounts:
      return self.accounts[account_number]
    else:
      raise ValueError("Account not found")


class Customer:
  def __init__(self, name: str, address: str, bank: Bank):
    self.name = name
    self.address = address
    self.bank = bank

  def open_account(self, initial_balance: float) -> Account:
    account_number = self._generate_account_number()
    account = self.bank.create_account(account_number, self.name, self.address, initial_balance)
    return account

  def _generate_account_number(self) -> int:
    return math.floor(random.random() * 1000000)


def banking_scenario():
  bank = Bank()
  customer1 = Customer("Alice", "Moscow, Stremyannyi per, 1", bank)
  customer2 = Customer("Bob", "Vorkuta, ul. Lenina, 5", bank)

  # Alice opens an account and deposits some money
  alice_account = customer1.open_account(initial_balance=500.0)
  alice_account.deposit(100.0)
  print(f"Alice's balance: {alice_account.balance}")  # Alice's balance: 600.0

  # Bob opens an account and deposits some money
  bob_account = customer2.open_account(initial_balance=1000.0)
  bob_account.deposit(500.0)
  print(f"Bob's balance: {bob_account.balance}")  # Bob's balance: 1500.0

  # Alice withdraws some money from her account
  alice_account.withdraw(300.0)
  print(f"Alice's balance: {alice_account.balance}")  # Alice's balance: 300.0

  # Alice tries to withdraw more money than she has in her account
  try:
    alice_account.withdraw(500.0)
  except ValueError as e:
    print(e)  # Insufficient funds

  # Bank retrieves Alice's account using the account number
  retrieved_account = bank.get_account(alice_account.account_number)
  print(f"Account {retrieved_account.account_number} by {retrieved_account.customer_name} ({retrieved_account.customer_address}), balance {retrieved_account.balance}") # Account XXXXXX by Alice (Moscow, Stremyannyi per, 1), balance 300.0
```

## SOLUTION:

### Code Issues Description

1. **Low Level of Abstraction**:
   - **Class `Account`**:
     - The `deposit` and `withdraw` methods interact directly with the `Bank` class attributes, violating encapsulation and leading to tight coupling between the classes.
   
2. **High Coupling**:
   - **Classes `Account` and `Bank`**:
     - The `Account` class depends on the `Bank` class to perform deposit and withdrawal operations. This results in high coupling, making it difficult to change and test the classes independently.
   
3. **Low Cohesion**:
   - **Class `Customer`**:
     - The account number generation in the `_generate_account_number` method is not tied to the bank and can create duplicate account numbers. Also, it makes more sense for the account number to be generated within the `Bank` class.
   
4. **Layout Problems**:
   - **Method `banking_scenario`**:
     - The client-bank interaction scenario is implemented in one function, making it hard to read and test. It is logical to split it into separate functions for each action (opening accounts, deposits, withdrawals, etc.).

### Improving Code Quality

To address the identified issues, the following strategy is proposed:

1. **Improving Abstraction**:
   - Move deposit and withdrawal operations to the `Bank` class.
   
2. **Reducing Coupling**:
   - Move the logic for interacting with the account balance to the `Bank` class, reducing the dependency between the `Account` and `Bank` classes.
   
3. **Increasing Cohesion**:
   - Move account number generation to the `Bank` class to ensure unique account numbers.
   
4. **Improving Layout**:
   - Split the `banking_scenario` method into smaller methods.

### Improved Version of the Code

```python
import math
import random

class Account:
  def __init__(self, account_number: int, customer_name: str, customer_address: str, balance: float):
    self.account_number = account_number
    self.balance = balance
    self.customer_name = customer_name
    self.customer_address = customer_address

class Bank:
  def __init__(self):
    self.accounts = {}

  def create_account(self, customer_name: str, customer_address: str, initial_balance: float) -> Account:
    account_number = self._generate_account_number()
    account = Account(account_number, customer_name, customer_address, initial_balance)
    self.accounts[account_number] = account
    return account

  def get_account(self, account_number: int) -> Account:
    if account_number in self.accounts:
      return self.accounts[account_number]
    else:
      raise ValueError("Account not found")

  def deposit(self, account_number: int, amount: float):
    if account_number in self.accounts:
      self.accounts[account_number].balance += amount
    else:
      raise ValueError("Account not found")

  def withdraw(self, account_number: int, amount: float):
    if account_number in self.accounts:
      if amount <= self.accounts[account_number].balance:
        self.accounts[account_number].balance -= amount
      else:
        raise ValueError("Insufficient funds")
    else:
      raise ValueError("Account not found")

  def _generate_account_number(self) -> int:
    while True:
      account_number = math.floor(random.random() * 1000000)
      if account_number not in self.accounts:
        return account_number

class Customer:
  def __init__(self, name: str, address: str, bank: Bank):
    self.name = name
    self.address = address
    self.bank = bank

  def open_account(self, initial_balance: float) -> Account:
    return self.bank.create_account(self.name, self.address, initial_balance)

def banking_scenario():
  bank = Bank()
  customer1 = Customer("Alice", "Moscow, Stremyannyi per, 1", bank)
  customer2 = Customer("Bob", "Vorkuta, ul. Lenina, 5", bank)

  # Alice opens an account and deposits some money
  alice_account = customer1.open_account(initial_balance=500.0)
  bank.deposit(alice_account.account_number, 100.0)
  print(f"Alice's balance: {alice_account.balance}")  # Alice's balance: 600.0

  # Bob opens an account and deposits some money
  bob_account = customer2.open_account(initial_balance=1000.0)
  bank.deposit(bob_account.account_number, 500.0)
  print(f"Bob's balance: {bob_account.balance}")  # Bob's balance: 1500.0

  # Alice withdraws some money from her account
  bank.withdraw(alice_account.account_number, 300.0)
  print(f"Alice's balance: {alice_account.balance}")  # Alice's balance: 300.0

  # Alice tries to withdraw more money than she has in her account
  try:
    bank.withdraw(alice_account.account_number, 500.0)
  except ValueError as e:
    print(e)  # Insufficient funds

  # Bank retrieves Alice's account using the account number
  retrieved_account = bank.get_account(alice_account.account_number)
  print(f"Account {retrieved_account.account_number} by {retrieved_account.customer_name} ({retrieved_account.customer_address}), balance {retrieved_account.balance}") # Account XXXXXX by Alice (Moscow, Stremyannyi per, 1), balance 300.0

if __name__ == "__main__":
  banking_scenario()
```

### Explanation of Improvements

1. **Improving Abstraction**:
   - Moved the `deposit` and `withdraw` methods to the `Bank` class, improving encapsulation and responsibility distribution.

2. **Reducing Coupling**:
   - Removed the dependency between the `Account` and `Bank

+++END_EN+++
+++START_RUS+++

## ЗАДАЧА:

**Изучите предоставленный пример кода и выделите в нем проблемы: низкий уровень абстракции, высокая связность, низкая цельность, проблемы с компоновкой (опишите в каких классах и функциях вы нашли каждую из проблем). Предложите способы улучшения качества кода в примере. Реализуйте улучшенную версию и объясните, каким образом вы решили выявленные проблемы.**

### Пример кода:

```python
import math
import random

class Account:
  def __init__(self, bank, account_number: int, customer_name: str, customer_address: str, balance: float):
    self.account_number = account_number
    self.balance = balance
    self.customer_name = customer_name
    self.customer_address = customer_address
    self.bank = bank

  def deposit(self, amount: float):
    self.bank.accounts[self.account_number].balance += amount

  def withdraw(self, amount: float):
    if amount <= self.bank.accounts[self.account_number].balance:
      self.bank.accounts[self.account_number].balance -= amount
    else:
      raise ValueError("Insufficient funds")


class Bank:
  def __init__(self):
    self.accounts = {}

  def create_account(self, account_number: int, customer_name: str, customer_address: str, initial_balance: float) -> Account:
    account = Account(self, account_number, customer_name, customer_address, initial_balance)
    self.accounts[account_number] = account
    return account

  def get_account(self, account_number: int) -> Account:
    if account_number in self.accounts:
      return self.accounts[account_number]
    else:
      raise ValueError("Account not found")


class Customer:
  def __init__(self, name: str, address: str, bank: Bank):
    self.name = name
    self.address = address
    self.bank = bank

  def open_account(self, initial_balance: float) -> Account:
    account_number = self._generate_account_number()
    account = self.bank.create_account(account_number, self.name, self.address, initial_balance)
    return account

  def _generate_account_number(self) -> int:
    return math.floor(random.random() * 1000000)


def banking_scenario():
  bank = Bank()
  customer1 = Customer("Alice", "Moscow, Stremyannyi per, 1", bank)
  customer2 = Customer("Bob", "Vorkuta, ul. Lenina, 5", bank)

  # Alice opens an account and deposits some money
  alice_account = customer1.open_account(initial_balance=500.0)
  alice_account.deposit(100.0)
  print(f"Alice's balance: {alice_account.balance}")  # Alice's balance: 600.0

  # Bob opens an account and deposits some money
  bob_account = customer2.open_account(initial_balance=1000.0)
  bob_account.deposit(500.0)
  print(f"Bob's balance: {bob_account.balance}")  # Bob's balance: 1500.0

  # Alice withdraws some money from her account
  alice_account.withdraw(300.0)
  print(f"Alice's balance: {alice_account.balance}")  # Alice's balance: 300.0

  # Alice tries to withdraw more money than she has in her account
  try:
    alice_account.withdraw(500.0)
  except ValueError as e:
    print(e)  # Insufficient funds

  # Bank retrieves Alice's account using the account number
  retrieved_account = bank.get_account(alice_account.account_number)
  print(f"Account {retrieved_account.account_number} by {retrieved_account.customer_name} ({retrieved_account.customer_address}), balance {retrieved_account.balance}") # Account XXXXXX by Alice (Moscow, Stremyannyi per, 1), balance 300.0
```

## РЕШЕНИЕ:

### Описание проблем кода

1. **Низкий уровень абстракции**:
   - **Класс `Account`**:
     - Методы `deposit` и `withdraw` напрямую взаимодействуют с атрибутами класса `Bank`, что нарушает инкапсуляцию и приводит к тесной связности между классами.
   
2. **Высокая связность**:
   - **Классы `Account` и `Bank`**:
     - Класс `Account` зависит от класса `Bank` для выполнения операций депозита и снятия средств. Это приводит к высокой связности, что усложняет изменение и тестирование классов отдельно.
   
3. **Низкая цельность**:
   - **Класс `Customer`**:
     - Генерация номера счета в методе `_generate_account_number` не привязана к банку и может создавать дубликаты номеров счетов. Также логично, чтобы номер счета генерировался внутри класса `Bank`.
   
4. **Проблемы с компоновкой**:
   - **Метод `banking_scenario`**:
     - Сценарий взаимодействия клиентов с банком реализован в одной функции, что делает его трудно читаемым и тестируемым. Логично разделить на отдельные функции для каждого действия (открытие счета, депозиты, снятие и т.д.).

### Улучшение качества кода

Для решения выявленных проблем предлагается следующая стратегия:

1. **Улучшение абстракции**:
   - Перенести операции депозита и снятия средств в класс `Bank`.
   
2. **Уменьшение связности**:
   - Переместить логику взаимодействия с балансом счета в класс `Bank`, чтобы уменьшить зависимость между классами `Account` и `Bank`.
   
3. **Повышение цельности**:
   - Генерацию номера счета перенести в класс `Bank`, что обеспечит уникальность номеров счетов.
   
4. **Улучшение компоновки**:
   - Разделить метод `banking_scenario` на более мелкие методы.

### Улучшенная версия кода

```python
import math
import random

class Account:
  def __init__(self, account_number: int, customer_name: str, customer_address: str, balance: float):
    self.account_number = account_number
    self.balance = balance
    self.customer_name = customer_name
    self.customer_address = customer_address

class Bank:
  def __init__(self):
    self.accounts = {}

  def create_account(self, customer_name: str, customer_address: str, initial_balance: float) -> Account:
    account_number = self._generate_account_number()
    account = Account(account_number, customer_name, customer_address, initial_balance)
    self.accounts[account_number] = account
    return account

  def get_account(self, account_number: int) -> Account:
    if account_number in self.accounts:
      return self.accounts[account_number]
    else:
      raise ValueError("Account not found")

  def deposit(self, account_number: int, amount: float):
    if account_number in self.accounts:
      self.accounts[account_number].balance += amount
    else:
      raise ValueError("Account not found")

  def withdraw(self, account_number: int, amount: float):
    if account_number in self.accounts:
      if amount <= self.accounts[account_number].balance:
        self.accounts[account_number].balance -= amount
      else:
        raise ValueError("Insufficient funds")
    else:
      raise ValueError("Account not found")

  def _generate_account_number(self) -> int:
    while True:
      account_number = math.floor(random.random() * 1000000)
      if account_number not in self.accounts:
        return account_number

class Customer:
  def __init__(self, name: str, address: str, bank: Bank):
    self.name = name
    self.address = address
    self.bank = bank

  def open_account(self, initial_balance: float) -> Account:
    return self.bank.create_account(self.name, self.address, initial_balance)

def banking_scenario():
  bank = Bank()
  customer1 = Customer("Alice", "Moscow, Stremyannyi per, 1", bank)
  customer2 = Customer("Bob", "Vorkuta, ul. Lenina, 5", bank)

  # Alice opens an account and deposits some money
  alice_account = customer1.open_account(initial_balance=500.0)
  bank.deposit(alice_account.account_number, 100.0)
  print(f"Alice's balance: {alice_account.balance}")  # Alice's balance: 600.0

  # Bob opens an account and deposits some money
  bob_account = customer2.open_account(initial_balance=1000.0)
  bank.deposit(bob_account.account_number, 500.0)
  print(f"Bob's balance: {bob_account.balance}")  # Bob's balance: 1500.0

  # Alice withdraws some money from her account
  bank.withdraw(alice_account.account_number, 300.0)
  print(f"Alice's balance: {alice_account.balance}")  # Alice's balance: 300.0

  # Alice tries to withdraw more money than she has in her account
  try:
    bank.withdraw(alice_account.account_number, 500.0)
  except ValueError as e:
    print(e)  # Insufficient funds

  # Bank retrieves Alice's account using the account number
  retrieved_account = bank.get_account(alice_account.account_number)
  print(f"Account {retrieved_account.account_number} by {retrieved_account.customer_name} ({retrieved_account.customer_address}), balance {retrieved_account.balance}") # Account XXXXXX by Alice (Moscow, Stremyannyi per, 1), balance 300.0

if __name__ == "__main__":
  banking_scenario()
```

### Объяснение улучшений

1. **Улучшение абстракции**:
   - Переместили методы `deposit` и `withdraw` в класс `Bank`, что улучшает инкапсуляцию и распределение ответственности.

2. **Уменьшение связности**:
   - Убрали зависимость между классами `Account` и `Bank`. Теперь класс `Account` не взаимодействует напрямую с балансом через класс `Bank`.

3. **Повышение цельности**:
   - Генерация номера счета перемещена в класс `Bank`, чтобы обеспечить уникальность номеров счетов.

4. **Улучшение компоновки**:
   - Метод `banking_scenario` теперь использует методы банка для выполнения операций, что делает его более читабельным и легким для тестирования.

+++END_RUS+++
4. **Улучшение компоновки**:
   - Метод `banking_scenario` теперь использует методы банка для выполнения операций, что делает его более читабельным и легким для тестирования.
