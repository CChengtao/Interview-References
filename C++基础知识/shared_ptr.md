```cpp
template<typename T>
class sharedPtr {
public:
	sharedPtr() {
		ptr = nullptr;
		useCount = nullptr;
	}
	sharedPtr(T* p) {
		ptr = p;
		try {
			useCount = new int(1);
		}
		catch (...) {
			delete ptr;
			ptr = nullptr;
			delete useCount;
			useCount = nullptr;
		}

	}
	sharedPtr(const sharedPtr<T>& origin) {
		this->useCount = origin.useCount;
		this->ptr = origin.ptr;
		++(*useCount);
	}
	sharedPtr<T>& operator=(const sharedPtr<T>& rhs) {
		++(*(rhs.useCount));
		if (--(*useCount) == 0) {
			delete ptr;
			ptr = nullptr;
			delete useCount;
			useCount = nullptr;
		}
		this->ptr = rhs.ptr;
		*useCount = *(rhs.useCount);
		return *this;
	}
	~sharedPtr() {
		if (--(*useCount) == 0) {
			getCount();
			delete ptr;
			ptr = nullptr;
			delete useCount;
			useCount = nullptr;
		}
	}
	int getCount() {
		return *useCount;
	}
	T operator*() {
		return *ptr;
	}
	T* operator->() {
		return ptr;
	}
private:
	T* ptr;
	int* useCount;
};
```

