Question 1: By default, are Django signals executed synchronously or
asynchronously? Please support your answer with a code snippet that conclusively
proves your stance. The code does not need to be elegant and production ready, we
just need to understand your logic.
Ans:
Django is a popular web framework known for its robustness, flexibility, 
and security.One of the features that make Django powerful is its signal system. Signals allow developers 
to trigger certain actions when specific events occur, such as when a model is saved or deleted.

The answer of the above question is Yes. Django signals are executed synchronously,
When a signal is triggered, the associated receivers are executed synchronously, meaning that each receiver must complete before the next one can begin.

# models.py
from django.db import models
from django.db.models.signals import post_save
from django.dispatch import receiver
import time

# A simple model for demonstration
class MyModel(models.Model):
    name = models.CharField(max_length=100)

# Signal receiver function
@receiver(post_save, sender=MyModel)
def my_signal_handler(sender, instance, **kwargs):
    print("Signal handler started")
    time.sleep(5)  # Simulate a long-running task
    print("Signal handler finished")



# Output:
 Signal handler started
 [5-second delay]
 Signal handler finished
 The create() operation will be delayed by the time.sleep(5) in the signal handler.

Synchronous Execution: The my_signal_handler function starts executing right after the model instance is saved and blocks further execution until it finishes. 
During this time, the Django shell or request is effectively stalled, which demonstrates synchronous execution. The delay introduced by time.sleep(5) means
that the instance creation in the shell waits for the signal handler to complete.
===============================================================================================================

Question 2: Do Django signals run in the same thread as the caller? Please support
your answer with a code snippet that conclusively proves your stance. The code does
not need to be elegant and production ready, we just need to understand your logic.
Ans:
Yes, Django signals run in the same thread as the caller. This means that when a 
signal is emitted, its signal handlers are executed in the same thread that emitted the signal.


# models.py

from django.db import models
from django.db.models.signals import post_save
from django.dispatch import receiver
import threading

# A simple model for demonstration
class MyModel(models.Model):
    name = models.CharField(max_length=100)

# Signal receiver function
@receiver(post_save, sender=MyModel)
def my_signal_handler(sender, instance, **kwargs):
    print(f"Signal handler running in thread: {threading.current_thread().name}")

# Function to create a model instance
def create_model_instance():
    print(f"Creating model instance in thread: {threading.current_thread().name}")
    MyModel.objects.create(name="Test")



# Output:
 Creating model instance in thread: TestThread
 Signal handler running in thread: TestThread

Signal Definition: We define a Django model MyModel and connect a signal handler to the post_save signal of MyModel. 
The signal handler my_signal_handler prints the name of the current thread.

Thread Creation: We define a function create_model_instance that creates an instance of MyModel.
This function is intended to be run in a separate thread.

Thread Execution: In the Django shell, we create and start a new thread to execute the create_model_instance function. 
After starting the thread, we wait for it to finish using thread.join().

Output Observation: When the model instance is created within the new thread, the signal handler prints the name of the thread it is running in. 
The output will show that both the model instance creation and the signal handling are happening in the same thread (in this case, 'TestThread').
==================================================================================================================================================

Question 3: By default, do Django signals run in the same database transaction as
the caller? Please support your answer with a code snippet that conclusively proves
your stance. The code does not need to be elegant and production ready, we just need
to understand your logic.
Ans:

By default, Django signals run in the same database transaction as the caller. 
This means that if a signal is triggered by a database operation (such as saving a model instance),
the signal handlers will execute within the same transaction as that operation. As a result, 
if the transaction is rolled back, the changes made by the signal handler will also be reverted.


# models.py

from django.db import models, transaction
from django.db.models.signals import post_save
from django.dispatch import receiver

# A simple model for demonstration
class MyModel(models.Model):
    name = models.CharField(max_length=100)

# Another model to demonstrate transaction rollback
class AuditLog(models.Model):
    message = models.CharField(max_length=255)

# Signal receiver function that creates an audit log entry
@receiver(post_save, sender=MyModel)
def create_audit_log(sender, instance, **kwargs):
    AuditLog.objects.create(message=f"Model {instance.name} was created.")

# Function to test transaction rollback
def test_transaction_rollback():
    try:
        with transaction.atomic():
            # Create a model instance
            MyModel.objects.create(name="Test")

            # At this point, an audit log entry should be created by the signal handler

            # Simulate an error that causes the transaction to rollback
            raise Exception("Triggering rollback")

    except Exception as e:
        print(f"Exception caught: {e}")

# Output:
 Exception caught: Triggering rollback
 The AuditLog table should be empty, showing that the signal handler's changes were rolled back.


