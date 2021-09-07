### WiredTiger 相關

- 記憶體使用

  - MongoDB 依賴於其 WiredTiger 內部快取及文件系統快取。
  - WiredTiger 內部快取儲存最近使用且未壓縮的頁，一般將從文件系統快取或硬碟經由解壓縮複製進來，在每次執行查詢時都會將一些頁複製進快取 (包含內部快取及文件系統快取)，以供後續更快地檢索數據。
  - WiredTiger 內部快取在預設配置下將使用 50% 的可用 RAM，可透過 `storage.wiredTiger.engineConfig.cacheSizeGB` 選項更改成指定大小。
  - 文件系統快取儲存最近使用且已壓縮的頁，可做為被壓縮到 WT 文件中的硬碟頁快取，當越多的硬碟頁被快取表示更大可能可直接從 RAM 訪問數據，而不是從延遲較高的硬碟。
  - 文件系統快取將自動使用所有未被 WiredTiger 內部快取或其它進程使用的空閒 RAM。