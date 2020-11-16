# 组合模式

组合模式主要用来处理树形结构数据

将一组对象组织成树形结构，以表示一种部分-整体的层次结构

组合模式让使用者可以统一单个对象和组合对象的处理逻辑，单一对象和组合对象实现相同的接口

```cpp
class FileNode {
protected:
    std::string name;

public:
    FileNode(const std::string& n) : name(n) {}
    virtual ~FileNode() {}

    virtual int GetNumOfFiles() = 0;
    virtual long GetSizeOfFile() = 0;

    const std::string& GetNameOfFile() {return name;}
}

class File : public FileNode {
public:
    File(std::string n) : FileNode(n) {}
    virtual ~File() {}

    int GetNumOfFiles() override {return 1;}
    long GetSizeOfFile() override {return 100;}
}

class Directory : public FileNode {
public:
    Directory(std::string n) : FileNode(n) {}
    virtual ~Directory() {}

    int GetNumOfFiles() override {
        int res = 0;
        for(const auto& it = files.begin(); it != files.end(); ++it) {
            res += it->GetNumOfFiles();
        }
        return res;
    }
    long GetSizeOfFile() override {
        int res = 0;
        for(const auto& it = files.begin(); it != files.end(); ++it) {
            res += it->GetSizeOfFile();
        }
        return res;
    }
    void Add(FileNode * node) {
        files.push_back(node);
    }
    void Remove(std::string n) {
        for(auto it = files.begin(); it != files.end(); ++it) {
            if(it->GetNameOfFile() == n) {
                files.erase(it);
                break;
            }
        }
    }

private:
    std::list<FileNode*> files;
}
```

组合对象的关键在于它定义了一个抽象构建类，它既可表示叶子对象，也可表示容器对象