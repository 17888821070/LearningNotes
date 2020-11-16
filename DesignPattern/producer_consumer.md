# 生产者/消费者模式

生产者和消费者是面向过程的编程模式

生产者和消费者问题是线程模型中的经典问题：生产者和消费者在同一时间段内共用同一个存储空间，生产者往存储空间中添加产品，消费者从存储空间中取走产品，当存储空间为空时，消费者阻塞，当存储空间满时，生产者阻塞

```cpp
struct Task {
	int num;
	Task(int n) : num(n) {}
	typedef shared_ptr<Task> ptr;
};

// 同步队列
class SyncQueue {
private:
	size_t max_size;
	list<Task::ptr> tasks;
	mutex mt;
	condition_variable cv_notempty;
	condition_variable cv_notfull;
	atomic<bool> flag;

public:
	SyncQueue(size_t size) : max_size(size), flag(true) {}
	~SyncQueue() {
		if (flag) {
			stop();
		}
	}

	void push(Task::ptr && ptr) {
		add(ptr);
	}

	Task::ptr take() {
		unique_lock<mutex> ul(mt);
		cv_notempty.wait(ul, [this]() {
			return !this->flag || this->notempty();
		});
		if (!flag) return nullptr;
		auto res = tasks.front();
		tasks.pop_front();
		return res;
	}

	void stop() {
		{
            // 缩短临界区
			lock_guard<mutex> lg(mt);
			flag = false;
		}
		cv_notempty.notify_all();
		cv_notfull.notify_all();
	}

private:
	bool notempty() {
		return !tasks.empty();
	}

	bool notfull() {
		return tasks.size() < max_size;
	}

	void add(Task::ptr & task) {
		unique_lock<mutex> ul(mt);
		cv_notfull.wait(ul, [this]() {
			return !this->flag || this->notfull();
		});
		if (!flag) return;
		tasks.push_back(task);
		cv_notempty.notify_one();
	}
};

class ThreadPool {
private:
	list<thread> ths;
	SyncQueue q;
	atomic<bool> flag;

private:
	void solve(Task::ptr task) {
		if (!task) return;
		cout << "this thread is: " << this_thread::get_id() <<", num = "<<
			task->num<<endl;
		this_thread::sleep_for(chrono::seconds(5));
	}

	void run() {
		while (flag) {
			auto task = q.take();
			solve(task);
		}
	}

public:
	ThreadPool() : q(200) {}
	~ThreadPool() {}

	void start(int num) {
		flag = true;
		for (int i = 0; i < num; ++i) {
			ths.emplace_back(thread(&ThreadPool::run, this));
		}
	}

	void add(Task::ptr&& task) {
		q.push(forward<Task::ptr>(task));
	}

	void stop() {
		q.stop();
		flag = false;
		for (auto& it : ths) {
			it.join();
		}
	}
};

int main() {
	ThreadPool tp;
	for (int i = 0; i < 100; ++i) {
		tp.add(make_shared<Task>(i));
	}
	tp.start(5);
	getchar();
	tp.stop();
	return 0;
}
```
