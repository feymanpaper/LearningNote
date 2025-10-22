```python
class SyncedEnforcer
	def __init__(self, model=None, adapter=None):
		self._e = Enforcer(model, adapter)
		self._rwlock = RWLockWrite()
		self._rl = self._rwlock.gen_rlock()
		self._wl = self._rwlock.gen_wlock()
		self._auto_loading = AtomicBool(False)
		self._auto_loading_thread = None

	def load_policy(self):
		"""reloads the policy from file/database."""
		with self._wl:
			return self._e.load_policy()
	
	def add_policy(self, *params):
		"""adds an authorization rule to the current policy.
		
		If the rule already exists, the function returns false and the rule will not be added.
		
		Otherwise the function returns true by adding the new rule.
		
		"""
		with self._wl:
			return self._e.add_policy(*params)
```

```python
  def _add_policy(self, sec, ptype, rule):
        """adds a rule to the current policy."""
        if self.model.has_policy(sec, ptype, rule):
            return False
        if self.adapter and self.auto_save:
            if self.adapter.add_policy(sec, ptype, rule) is False:
                return False
            if self.watcher and self.auto_notify_watcher:
                if callable(getattr(self.watcher, "update_for_add_policy", None)):
                    self.watcher.update_for_add_policy(sec, ptype, rule)
                else:
                    self.watcher.update()
        rule_added = self.model.add_policy(sec, ptype, rule)
```

```python
  def update(self):
        def func():
            with self.mutex:
                msg = MSG("Update", self.options.local_ID, "", "", "")
                return self.pub_client.publish(self.options.channel, msg.marshal_binary())

        return self.log_record(func)
```


```python
    def subscribe(self):
        time.sleep(self.sleep)
        try:
            if not self.sub_client:
                rds = self._get_redis_conn()
                self.sub_client = rds.client().pubsub()
            self.sub_client.subscribe(self.options.channel)
            print(f"Waiting for casbin updates... in the worker: {self.options.local_ID}")
            if self.execute_update:
                self.update()
            try:
                for item in self.sub_client.listen():
                    if not self.subscribe_event.is_set():
                        self.subscribe_event.set()
                    if item is not None and item["type"] == "message":
                        try:
                            with self.mutex:
                                self.callback(str(item))
                        except Exception as listen_exc:
                            print(
                                "Casbin Redis watcher failed sending update to teh callback function "
                                " process due to: {}".format(str(listen_exc))
                            )
                            if self.sub_client:
                                self.sub_client.close()
                            break
            except Exception as sub_exc:
                print("Casbin Redis watcher failed to get message from redis due to {}".format(str(sub_exc)))
                if self.sub_client:
                    self.sub_client.close()
        except Exception as redis_exc:
            print("Casbin Redis watcher failed to subscribe due to: {}".format(str(redis_exc)))
        finally:
            if self.sub_client:
                self.sub_client.close()
```