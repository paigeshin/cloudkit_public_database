# cloudkit_public_database

```swift
import CloudKit
import UIKit

class ViewController: UIViewController, UITableViewDataSource {

    let tableView: UITableView = {
        let tableView = UITableView()
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        return tableView
    } ()
    
    private let database = CKContainer(identifier: "iCloud.com.paigesoftware.GroceryStore").publicCloudDatabase
    
    var items = [String]()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Grocery List"
        view.addSubview(self.tableView)
        tableView.dataSource = self
        let control = UIRefreshControl()
        control.addTarget(self, action: #selector(pullToRefresh), for: .valueChanged)
        tableView.refreshControl = control
        navigationItem.rightBarButtonItem = UIBarButtonItem(barButtonSystemItem: .add,
                                                            target: self,
                                                            action: #selector(didTapAdd))
        fetchItems()
    }
    
    @objc
    func pullToRefresh() {
        print("Fetch Items!!!")
        tableView.refreshControl?.beginRefreshing()
        let query = CKQuery(recordType: "GroceryItem",
                            predicate: NSPredicate(value: true))
//        database.fetch(withQuery: query) { result in
//            switch result {
//            case .success(let records):
//                DispatchQueue.main.async { [weak self] in
//                    self?.items = records.matchResults.compactMap { $0.0.value(forKey: "name") as? String }
//                    self?.tableView.reloadData()
//                }
//            case .failure(let error):
//                print(error.localizedDescription)
//            }
//        }
        database.perform(query, inZoneWith: nil) { [weak self] records, error in
            if let error {
                print(error.localizedDescription)
                return
            }
            if let records {
                DispatchQueue.main.async {
                    self?.items = records.compactMap { $0.value(forKey: "name") as? String }
                    self?.tableView.reloadData()
                    self?.tableView.refreshControl?.endRefreshing()
                }
            }
        }
    }
    
    func fetchItems() {
        print("Fetch Items!!!")
        let query = CKQuery(recordType: "GroceryItem",
                            predicate: NSPredicate(value: true))
//        database.fetch(withQuery: query) { result in
//            switch result {
//            case .success(let records):
//                DispatchQueue.main.async { [weak self] in
//                    self?.items = records.matchResults.compactMap { $0.0.value(forKey: "name") as? String }
//                    self?.tableView.reloadData()
//                }
//            case .failure(let error):
//                print(error.localizedDescription)
//            }
//        }
        database.perform(query, inZoneWith: nil) { [weak self] records, error in
            if let error {
                print(error.localizedDescription)
                return
            }
            if let records {
                DispatchQueue.main.async {
                    self?.items = records.compactMap { $0.value(forKey: "name") as? String }
                    self?.tableView.reloadData()
                }
            }
        }
    }
    
    @objc func didTapAdd() {
        let alert = UIAlertController(title: "Add Item", message: nil, preferredStyle: .alert)
        alert.addTextField { field in
            field.placeholder = "Enter Name..."
        }
        alert.addAction(UIAlertAction(title: "Cancel", style: .cancel, handler: nil))
        alert.addAction(UIAlertAction(title: "Add", style: .default, handler: { [weak self] _ in
            if let field = alert.textFields?.first, let text = field.text, !text.isEmpty {
                self?.saveItem(name: text)
            }
        }))
        present(alert, animated: true)
    }
    
    @objc func saveItem(name: String) {
        let record = CKRecord(recordType: "GroceryItem")
        record.setValue(name, forKey: "name")
        database.save(record) { [weak self] record, error in
            if let error {
                print(error.localizedDescription)
                return
            }
            if let record {
                print(record)
                DispatchQueue.main.async {
                    self?.fetchItems()
                }
            }
        }
    }

    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        self.tableView.frame = view.bounds
    }
    
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return items.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell",
                                                 for: indexPath)
        cell.textLabel?.text = items[indexPath.row]
        return cell
    }

}
```
