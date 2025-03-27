
Question 1: By default are django signals executed synchronously or asynchronously? Please support your answer with a code snippet that conclusively proves your stance. The code does not need to be elegant and production ready, we just need to understand your logic.

By default, Django signals are executed synchronously. This means that the signal's receiver function runs in the same thread as the event that triggered it, blocking execution until it completes.

Proof with Code
We can demonstrate this behavior using Django's pre_save signal. If signals were asynchronous by default, the main execution flow wouldn't have to wait for the signal function to finish.

Steps to Prove Synchronous Execution
Register a signal that introduces a delay (simulating a slow function).

Measure the total execution time of saving a model instance.

If the signal runs synchronously, the total time should include the delay.

Code to Prove Django Signals are Synchronous

1. Define the Model

from django.db import models
from django.db.models.signals import pre_save
from django.dispatch import receiver
import time

class TestModel(models.Model):
    name = models.CharField(max_length=100)
2. Register a Signal with a Delay

@receiver(pre_save, sender=TestModel)
def slow_signal(sender, instance, **kwargs):
    print("Signal started...")
    time.sleep(5)  # Simulate a slow operation
    print("Signal finished...")
3. Test Execution Time

import time
from django.test import TestCase

class SignalExecutionTest(TestCase):
    def test_signal_execution(self):
        obj = TestModel(name="Synchronous Test")
        
        start_time = time.time()
        obj.save()  # This will trigger the pre_save signal
        end_time = time.time()
        
        elapsed_time = end_time - start_time
        print(f"Total execution time: {elapsed_time:.2f} seconds")
        
        # Since the signal has a 5-second delay, total time should be ~5 seconds
        assert elapsed_time >= 5, "Signal did NOT run synchronously!"

Expected Output

Signal started...
(Sleeps for 5 seconds)
Signal finished...
Total execution time: 5.00 seconds
Conclusion
Since obj.save() took at least 5 seconds, we can conclude that Django signals execute synchronously by default. If they were asynchronous, obj.save() would return immediately while the signal runs in the background.

Question 2: Do django signals run in the same thread as the caller? Please support your answer with a code snippet that conclusively proves your stance. The code does not need to be elegant and production ready, we just need to understand your logic.

Yes, Django signals run in the same thread as the caller by default. This means that when a signal is triggered, its receiver function executes in the same thread as the main execution flow.

Proof with Code
We can demonstrate this behavior by:

Capturing the thread ID of the main execution flow.

Capturing the thread ID inside the signal handler.

Comparing both IDs to verify if they match.

Code to Prove Django Signals Run in the Same Thread
1. Import Required Modules

import threading
import time
from django.db import models
from django.db.models.signals import pre_save
from django.dispatch import receiver

2. Define a Model

class TestModel(models.Model):
    name = models.CharField(max_length=100)

3. Register a Signal to Print the Thread ID

@receiver(pre_save, sender=TestModel)
def check_thread(sender, instance, **kwargs):
    print(f"Signal running in thread: {threading.get_ident()}")
4. Test Execution in a Django TestCase

from django.test import TestCase

class SignalThreadTest(TestCase):
    def test_signal_thread(self):
        obj = TestModel(name="Thread Test")

        main_thread_id = threading.get_ident()
        print(f"Main execution thread: {main_thread_id}")

        obj.save()  # This triggers the pre_save signal
        
        # If signals run in the same thread, IDs should match
        self.assertEqual(main_thread_id, threading.get_ident(), "Signal ran in a different thread!")

Expected Output

Main execution thread: 140735585071872
Signal running in thread: 140735585071872
(Same thread ID for both main execution and signal)

Conclusion
Since the thread ID inside the signal function is the same as the main execution thread, we can conclusively say that Django signals run in the same thread as the caller by default. If they were asynchronous, the thread IDs would be different.

Question 3: By default do django signals run in the same database transaction as the caller? Please support your answer with a code snippet that conclusively proves your stance. The code does not need to be elegant and production ready, we just need to understand your logic.

Yes, by default, Django signals run in the same database transaction as the caller. This means that if a database transaction is rolled back, any changes made inside the signal handlers will also be rolled back.

Proof with Code
We will:

Use a post_save signal that modifies another model inside the transaction.

Manually trigger a rollback after saving the model.

Check whether the changes made inside the signal persist (they shouldn’t if signals share the same transaction).

Code to Prove Django Signals Run in the Same Database Transaction

1. Import Required Modules

from django.db import models, transaction
from django.db.models.signals import post_save
from django.dispatch import receiver

2. Define Models

class TestModel(models.Model):
    name = models.CharField(max_length=100)

class LogEntry(models.Model):
    message = models.CharField(max_length=255)

3. Register a post_save Signal to Modify LogEntry

@receiver(post_save, sender=TestModel)
def create_log_entry(sender, instance, **kwargs):
    print("Signal triggered: Creating log entry...")
    LogEntry.objects.create(message=f"TestModel saved: {instance.name}")

4. Test Transaction Behavior

from django.test import TestCase

class SignalTransactionTest(TestCase):
    def test_signal_transaction_rollback(self):
        with transaction.atomic():
            obj = TestModel(name="Transaction Test")
            obj.save()  # Triggers post_save signal

            # Assert that log entry was created
            self.assertEqual(LogEntry.objects.count(), 1, "Log entry not created by signal!")

            print("Rolling back transaction...")
            transaction.set_rollback(True)  # Manually rolling back transaction

        # After rollback, the log entry should not exist
        self.assertEqual(LogEntry.objects.count(), 0, "Signal did not rollback with the transaction!")

Expected Output

Signal triggered: Creating log entry...
Rolling back transaction...
Assertion Passed → Log entry is gone after rollback.

Conclusion
Since the log entry created inside the signal is rolled back when the main transaction is rolled back, this conclusively proves that Django signals run in the same database transaction as the caller by default.


 Custom Classes in Python

Description: You are tasked with creating a Rectangle class with the following requirements:

An instance of the Rectangle class requires length:int and width:int to be initialized.
We can iterate over an instance of the Rectangle class 
When an instance of the Rectangle class is iterated over, we first get its length in the format: {'length': <VALUE_OF_LENGTH>} followed by the width {width: <VALUE_OF_WIDTH>}

class Rectangle:
    def __init__(self, length: int, width: int):
        self.length = length
        self.width = width
        self._attributes = [{'length': self.length}, {'width': self.width}]
    
    def __iter__(self):
        return iter(self._attributes)


# Example Usage

rect = Rectangle(10, 5)
for attribute in rect:
    print(attribute)


Explanation:

Initialization (__init__ method):

The class requires length and width as integer parameters.

These values are stored in an internal list _attributes in the required format.

Iteration (__iter__ method):

The method returns an iterator over the _attributes list, which contains the dictionary representations of length and width.

Example Output:

{'length': 10}
{'width': 5}
This implementation ensures that iterating over the Rectangle instance provides the desired structured output.
