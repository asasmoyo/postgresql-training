Managing PostgreSQL Service
===========================

1. Checking service status

    ``` bash
    sudo pg_ctlcluster 13 main status
    ```

2. Starting service

    ``` bash
    sudo pg_ctlcluster 13 main start
    ```

3. Stopping service

    ``` bash
    sudo pg_ctlcluster 13 main stop
    ```

4. Restarting serice

    ``` bash
    sudo pg_ctlcluster 13 main restart
    ```

5. Tell PostgreSQL service to start after server boots

    ``` bash
    sudo systemctl enable postgresql
    sudo systemctl enable postgresql@13-main
    ```

Notice the `13` and `main`, they specify which cluster version and cluster folder that you want to manage.
