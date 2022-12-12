# MQTT-C Analysis Report

## Vulnerabilities

Vulnerabilities for **mqtt.c**

| Number | Severity | Issue                                                                          | Line                               |
| ------ | -------- | ------------------------------------------------------------------------------ | ---------------------------------- |
| 1.     | Critical | The Cognitive Complexity is too high compared to the 25 allow                  | L507; L649                         |
| 2.     | Critical | Nest more than 3 [if, for, do, while, switch] statements                       | L544; L736; L736; L750; L757; L762 |
| 3.     | Critical | Remove use of ellipsis notation.                                               | L1473; L1555                       |
| 4.     | Major    | The function has a too high number of parameters compared to the 7 authorized. | L233; L1105                        |
| 5.     | Major    | Have to merge the  "if" statement with the enclosing one.                      | L531; L1691                        |
| 6.     | Major    | Useless parentheses                                                            | L1669                              |
| 7.     | Minor    | Change the type of the variable to a pointer-to-const                          | L93; L887;  L1346;  L1641          |
| 8.     | Minor    | Variable should be declared inside the loop                                    | L102; 1689                         |
| 9.     | Minor    | Reduntant cast                                                                 | L160                               |

## Remediation

### 1. Cognitive Complexity (2 cases)

It concerned the functions ```ssize_t __mqtt_send(struct mqtt_client *client)``` and ```ssize_t __mqtt_recv(struct mqtt_client *client)```. This complexity comes from the number of ```if, else if, else, for, while, switch``` but also the number of nest.

The issue with a high cognitive complexity is that the code will be more difficulte to understand and so to maitain. If a bug or a secrurity breach were to be discovered, it would take more time to resolve it. A solution to reduce the complexity could be to create separated functions in order to divide thoses function into multiple parts.  

### 2. Nest of more than 3 statement (6 cases)

This issue is located in many part of the code, a nest is an accumulation of statement like ```if```, here is an exemple of a nest of 3:

```c
if{
    //code
    if{
    //code
        if{
        //code
        }
    }
}
```

Nest like this one are difficult to read and to understand. Similary to the first issue, the code will be harder to maintain

### 3. Ellipsis notation (2 cases)

Issues of ellipsis notation are in the ```ssize_t mqtt_pack_subscribe_request(uint8_t *buf, size_t bufsz, unsigned int packet_id, ...)``` and ```ssize_t mqtt_pack_unsubscribe_request(uint8_t *buf, size_t bufsz, unsigned int packet_id, ...)``` functions. The ```...``` at the end means that an undifined number of argument can be input. For exemple

```c
double average(int count, ...){
    //code
}

int main()
{
    // Function call
    double avg = average(6, 1, 2, 3, 4, 5, 6);
    return 0;
}
```

So average can have 2, 3, 7 or more argument. First it bypasses the compiler type checking. Also it can lead to disfunctionment of the program or even to attack.  A solution could be to replace the argument by a list of values.

### 4. Too high number of parameters (2 cases)

According to sonarqube a maximum of 7 arguments can be used, according to the C guidline, the maximum should be 4. This can lead to 2 issues : 

- Componenet values are no longer protected by an enforced invariant which leads to errors

- The function is probably doing more than one job violating the "one function, one responsibility rule"

The solution would be to fractione the function into 2 or more. Also, objects could be created contening thoses parameters.

### 5. Unmerdge if statement (2 cases)

Two if statement could be merge and increase the code readability. For exemple

```c
x = 3;
if (x > 0){
    if(x < 10){
        printf("{} is a digit", x)
    }
}
```

Would become

```c
x = 3;
if (x >= 0 && x < 10){
    printf("{} is a digit", x)
}
```

### 6. Useless parantheses (1 case)

This issue is in this line: 

```c
memmove(mqtt_mq_get(mq, new_tail_idx), mq->queue_tail, sizeof(struct mqtt_queued_message) * (size_t) ((new_tail_idx + 1

)));
```

The ```((new_tail_idx + 1)));``` should be written ```(new_tail_idx + 1));``` in order to avoid any miscomprehension of the code.

### 7. Type of the variable (4 cases)

For exemple the ```struct mqtt_queued_message *curr``` should be declared like this ```const struct mqtt_queued_message *curr```.  It allows to protect the object pointed and avoid any non-intented usage. It gives a better security to the program.

### 8. Variable should be declared inside the loop (2 cases)

The variable should be declared inside the loop, for exemple in 

```c
struct mqtt_queued_message *curr;
//code
    
for(curr = mqtt_mq_get(&(client->mq), 0); curr >= client->mq.queue_tail; --curr) {
// code
}
```

the code should be written like :

```c
```c
for(struct mqtt_queud_message *curr = mqtt_mq_get(&(client->mq), 0); curr >= client->mq.queue_tail; --curr) {
// code
}
```

It is a better way to write the code for 3 differents reasons:

- A better readability

- The variable can't be accenditely reused outside of the loop

- Resources are not retained longer than necessary

### 9. Reduntant cast (1 case)

Reductant cast make the code harder to read and understant. The declaration : 

```c
void* dest = (unsigned char*)client->recv_buffer.mem_start;
```

Should be changed by

```c
void* dest = client->recv_buffer.mem_start;
```
