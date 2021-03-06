### WiredTiger 相關

- 記憶體使用

  - MongoDB 依賴於其 WiredTiger 內部快取及文件系統快取。
  - WiredTiger 內部快取儲存最近使用且未壓縮的頁，一般將從文件系統快取或硬碟經由解壓縮複製進來，在每次執行查詢時都會將一些頁複製進快取 (包含內部快取及文件系統快取)，以供後續更快地檢索數據。
  - WiredTiger 內部快取在預設配置下將使用 50% 的可用 RAM，可透過 `storage.wiredTiger.engineConfig.cacheSizeGB` 選項更改成指定大小。
  - 文件系統快取儲存最近使用且已壓縮的頁，可做為被壓縮到 WT 文件中的硬碟頁快取，當越多的硬碟頁被快取表示更大可能可直接從 RAM 訪問數據，而不是從延遲較高的硬碟。
  - 文件系統快取將自動使用所有未被 WiredTiger 內部快取或其它進程使用的空閒 RAM。
  - 在同一台機器上運行多個 mongod 是不推薦的做法，不僅造成文件系統快取需額外的分配，還可能發生由於某個 mongod 使用過多資源導致其它 mongod 連帶影響性能問題。
  - 在大多數情況下，不建議將 WiredTiger 內部快取增加到高於預設 50% 的水平，雖然更多的查詢可從更大的快取中受益，但相對文件系統快取的減少將導致更多實際訪問硬碟的情況發生，從而拖慢整個資料庫速度。
