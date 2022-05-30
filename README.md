# ROS2-E2E-Evaluation

It is a ROS 2 performance framework for ROS 2 applications, such as Autoware.

ROS 2 version: foxy and galactic

## nif change notes

### ros2_tracing
Update from `galactic` branch of `ros2_tracing` to `4.1.0` (which is working, apparently), porting changes.

```bash
$ diff -q -r <ros2_tracing:galactic> <aitoware_perf/ros2_tracing>
...
Files ./tracetools/include/tracetools/tp_call.h and <aitoware_perf/ros2_tracing>/tracetools/include/tracetools/tp_call.h differ
Files ./tracetools/include/tracetools/tracetools.h and <aitoware_perf/ros2_tracing>/tracetools/include/tracetools/tracetools.h differ
Files ./tracetools/src/tracetools.c and <aitoware_perf/ros2_tracing>/tracetools/src/tracetools.c differ
...
Files ./tracetools_trace/tracetools_trace/tools/names.py and <aitoware_perf/ros2_tracing>/tracetools_trace/tracetools_trace/tools/names.py differ
```

Changes per-file (diff are enclosed into `rei >>>>>>>>>>>>>>>>>>>>>>>>>>>>.` brackets).

**tp_call.h**:
```c
TRACEPOINT_EVENT(
  TRACEPOINT_PROVIDER,
  rcl_lifecycle_transition,
  TP_ARGS(
    const void *, state_machine_arg,
    const char *, start_label_arg,
    const char *, goal_label_arg
  ),
  TP_FIELDS(
    ctf_integer_hex(const void *, state_machine, state_machine_arg)
    ctf_string(start_label, start_label_arg)
    ctf_string(goal_label, goal_label_arg)
  )
)

//rei >>>>>>>>>>>>>>>>>>>>>>>>>>>>.
TRACEPOINT_EVENT(
    TRACEPOINT_PROVIDER,
    rclcpp_subscribe,
    TP_ARGS(
        const void *, callback_arg,
        const uint64_t, source_stamp_arg,
        const uint64_t, received_stamp_arg
    ),
    TP_FIELDS(
        ctf_integer_hex(const void *, callback, callback_arg)
        ctf_integer(const uint64_t, source_stamp, source_stamp_arg)
        ctf_integer(const uint64_t, received_stamp, received_stamp_arg)
    )
)

TRACEPOINT_EVENT(
    TRACEPOINT_PROVIDER, client_request,
    TP_ARGS(
      const void *, client_arg,
      const void *, request_arg
    ),
    TP_FIELDS(
      ctf_integer_hex(const void *, client, client_arg)
      ctf_integer_hex(const void *, request, request_arg)
    )
)

TRACEPOINT_EVENT(
    TRACEPOINT_PROVIDER, service_request,
    TP_ARGS(
        const void *, service_arg,
        const uint64_t, source_stamp_arg,
        const uint64_t, received_stamp_arg
    ),
    TP_FIELDS(
        ctf_integer_hex(const void *, service, service_arg)
        ctf_integer(const uint64_t, source_stamp, source_stamp_arg)
        ctf_integer(const uint64_t, received_stamp, received_stamp_arg)
    )
)

TRACEPOINT_EVENT(
    TRACEPOINT_PROVIDER, service_response,
    TP_ARGS(
      const void *, service_arg,
      const void *, response_arg
    ),
    TP_FIELDS(
      ctf_integer_hex(const void *, service, service_arg)
      ctf_integer_hex(const void *, response, response_arg)
    )
)

TRACEPOINT_EVENT(
    TRACEPOINT_PROVIDER, client_response,
    TP_ARGS(
        const void *, client_arg,
        const uint64_t, source_stamp_arg,
        const uint64_t, received_stamp_arg
    ),
    TP_FIELDS(
        ctf_integer_hex(const void *, client, client_arg)
        ctf_integer(const uint64_t, source_stamp, source_stamp_arg)
        ctf_integer(const uint64_t, received_stamp, received_stamp_arg)
    )
)
//rei >>>>>>>>>>>>>>>>>>>>>>>>>>>>.

#endif  // _TRACETOOLS__TP_CALL_H_
```

**tracetools.h**:
```c
DECLARE_TRACEPOINT(
  rcl_lifecycle_transition,
  const void * state_machine,
  const char * start_label,
  const char * goal_label)

//rei >>>>>>>>>>>>>>>>>>>>>>>>>>>>.
DECLARE_TRACEPOINT(
  rclcpp_subscribe,
  const void *callback,
  const uint64_t source_stamp,
  const uint64_t received_stamp
)

DECLARE_TRACEPOINT(
  client_request, 
  const void *client,
  const void *request)

DECLARE_TRACEPOINT(
  service_request,
  const void *service,
  const uint64_t source_stamp,
  const uint64_t received_stamp
)

DECLARE_TRACEPOINT(
  service_response, 
  const void *service,
  const void *response)
  
DECLARE_TRACEPOINT(
  client_response,
  const void *client,
  const uint64_t source_stamp,
  const uint64_t received_stamp
)
//rei >>>>>>>>>>>>>>>>>>>>>>>>>>>>.

#ifdef __cplusplus
```

**tracetools.c**:
```c
void TRACEPOINT(
  rcl_lifecycle_transition,
  const void * state_machine,
  const char * start_label,
  const char * goal_label)
{
  CONDITIONAL_TP(
    rcl_lifecycle_transition,
    state_machine,
    start_label,
    goal_label);
}

//rei >>>>>>>>>>>>>>>>>>>>>>>>>>>>.
void TRACEPOINT(
    rclcpp_subscribe,
    const void *callback,
    const uint64_t source_stamp,
    const uint64_t received_stamp) {
  CONDITIONAL_TP(
      rclcpp_subscribe,
      callback,
      source_stamp,
      received_stamp
  );
}

void TRACEPOINT(
  client_request,
  const void *client,
  const void *request)
{
  CONDITIONAL_TP(
    client_request,
    client,
    request);
}

void TRACEPOINT(
    service_request,
    const void *service,
    const uint64_t source_stamp,
    const uint64_t received_stamp) {
  CONDITIONAL_TP(
      service_request,
      service,
      source_stamp,
      received_stamp
  );
}

void TRACEPOINT(
  service_response,
  const void *service,
  const void *response)
{
  CONDITIONAL_TP(
    service_response,
    service,
    response);
}

void TRACEPOINT(
    client_response,
    const void *client,
    const uint64_t source_stamp,
    const uint64_t received_stamp) {
  CONDITIONAL_TP(
      client_response,
      client,
      source_stamp,
      received_stamp
  );
}
//rei >>>>>>>>>>>>>>>>>>>>>>>>>>>>.

#ifndef _WIN32
```


**names.py (tracepoints.py)**:
```python
DEFAULT_EVENTS_ROS = [
    'ros2:rcl_init',
    'ros2:rcl_node_init',
    'ros2:rcl_publisher_init',
    'ros2:rcl_publish',
    'ros2:rclcpp_publish',
    'ros2:rcl_subscription_init',
    'ros2:rclcpp_subscription_init',
    'ros2:rclcpp_subscription_callback_added',
    'ros2:rcl_service_init',
    'ros2:rclcpp_service_callback_added',
    'ros2:rcl_client_init',
    'ros2:rcl_timer_init',
    'ros2:rclcpp_timer_callback_added',
    'ros2:rclcpp_timer_link_node',
    'ros2:rclcpp_callback_register',
    'ros2:callback_start',
    'ros2:callback_end',
    'ros2:rcl_lifecycle_state_machine_init',
    'ros2:rcl_lifecycle_transition',
    #rei >>>>>>>>>>>>>>>>>>>>>>>>>>>>.
    'ros2:rclcpp_subscribe',
    'ros2:client_request',
    'ros2:service_request',
    'ros2:service_response',
    'ros2:client_response',
    #rei >>>>>>>>>>>>>>>>>>>>>>>>>>>>.
]
```