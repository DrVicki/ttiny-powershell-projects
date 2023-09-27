# #1: Arrays are Immutable

Using PowerShell, you may find yourself in situations where you need to use an array, but did you know that arrays are immutable?

When we say that something is "immutable" in programming, we mean that its state or value cannot be changed after it is created.

For arrays in PowerShell, this immutability means that once an array is created, its size (the number of elements) is fixed. You cannot add or remove elements from an existing array.

For example, let's say you create an array in PowerShell like this:

```
$array = 1, 2, 3
```
This array has three elements. If you wanted to add a fourth element, you can't simply "push" a new element onto the end of the array. Instead, you would have to create a new array that includes the original three elements plus the new one:

```
$array = $array + 4
```
In this case, `$array + 4` creates a new array with four elements. 
  - The original three-element array is discarded, and the $array variable is updated to reference the new four-element array.

This immutability is in contrast to other data structures, like `ArrayLists` or `Lists`, which are mutable. 
  - With mutable data structures, you can add or remove elements without needing to create a new object.

In many cases where you'd normally use an array but need to be able to change the size (adding or removing elements), you could use a .NET  `ArrayList`. 
  - The `ArrayList` is a dynamic array, so you can add or remove elements.
  - Here's how you can create an `ArrayLis`t and add elements to it:

```
# Create an ArrayList
$list = New-Object System.Collections.ArrayList

# Add elements
$null = $list.Add("element1")
$null = $list.Add("element2")
$null = $list.Add("element3")
```
Note the `$null = portion` in each `Add` statement. 
  - Add method of   ArrayList   returns the index of the added element, and this is to suppress that output.

This tiny project is meant to give you an into to   ListArrays   and here is a full CRUD:

**Create an ArrayList and add items to it (Create):**

```
# Create an ArrayList
$list = New-Object System.Collections.ArrayList

# Add elements
$null = $list.Add("element1")
$null = $list.Add("element2")
$null = $list.Add("element3")
```
**Access elements from the `ArrayList` (Read):**

```
# Access an item
Write-Host "Item at index 1: " $list[1]

# Iterate over all items
foreach ($item in $list) {
    Write-Host $item
}
```
**Change an item in the `ArrayList` (Update):**

```
# Update an item
$list[1] = "newElement2"

# Verify the update
Write-Host "Updated item at index 1: " $list[1]
```
**Remove an item from the `ArrayList` (Delete):**

```
# Remove an item
$list.Remove("element3")

# Verify the removal
Write-Host "After removal:"
foreach ($item in $list) {
    Write-Host $item
}
```
**Note** that `ArrayLists` offer other useful methods and properties not shown here, such as Insert, Sort, Reverse and others. You can look up these methods and properties in the .NET Documentation.

## Armed with this information, let's refactor the following code to use an ArrayList


```
$source = Import-Csv -Path ./original.csv
$destination = Import-Csv -Path ./original.csv
$didntTransferOver = @()
$transferedPartiallyFirstNameNotMatching = @()
$transferedPartiallyLastNameNotMatching = @()
$transferedPartiallyPasswordNotMatching = @()
$fullyTransfered = @()


# loop over the source records
foreach ($item in $source) {
    # for every single record in source try to fid a matching record in destination
    $matchedItem = $destination | Where-Object { $_.email -eq $item.email }
    # if a matching record exist
    if($matchedItem){
        # Check whether other fields are matching 
        if($item.first_name -ne $matchedItem.first_name){
            # first name is not matching
            $transferedPartiallyFirstNameNotMatching += $matchedItem
        }
        elseif($item.last_name -ne $matchedItem.last_name){
            # last name is not matching
            $transferedPartiallyLastNameNotMatching += $matchedItem
        }
        elseif($item.encrypted_password -ne $matchedItem.encrypted_password){
            # encrypted password is not matching
            $transferedPartiallyPasswordNotMatching += $matchedItem
        } else {
            # everything is matching
            $fullyTransfered += $matchedItem
        }
    } else {
        # A matching record doesnt exist
        $didntTransferOver += $item
    }

}
$didntTransferOver | Export-Csv -Path ./didntTransferOver.csv  -NoTypeInformation
$transferedPartiallyFirstNameNotMatching | Export-Csv -Path ./transferedPartiallyFirstNameNotMatching.csv  -NoTypeInformation
$transferedPartiallyLastNameNotMatching | Export-Csv -Path ./transferedPartiallyLastNameNotMatching.csv  -NoTypeInformation
$transferedPartiallyPasswordNotMatching | Export-Csv -Path ./transferedPartiallyPasswordNotMatching.csv  -NoTypeInformation
$fullyTransfered  | Export-Csv -Path ./fullyTransfered.csv  -NoTypeInformation
```
This was a fivver project that wanted to know after an ETL job how many records moved over and check on the integrity of the records. 
  - **Here is the refactored version:**


```
# import the source and destination records
$source = Import-Csv -Path ./original.csv
$destinationRecords = Import-Csv -Path ./destination.csv

# Create a hashtable for destination records for faster lookup
$destination = @{}
$destinationRecords | ForEach-Object { $destination[$_.email] = $_ }

$didntTransferOver = New-Object System.Collections.ArrayList
$transferedPartiallyFirstNameNotMatching = New-Object System.Collections.ArrayList
$transferedPartiallyLastNameNotMatching = New-Object System.Collections.ArrayList
$transferedPartiallyPasswordNotMatching = New-Object System.Collections.ArrayList
$fullyTransfered = New-Object System.Collections.ArrayList

# loop over the source records
foreach ($item in $source) {
    $matchedItem = $destination[$item.email]

    if ($null -eq $matchedItem) {
        $didntTransferOver.Add($item) | Out-Null
    } else {
        if ($item.first_name -ne $matchedItem.first_name) {
            $transferedPartiallyFirstNameNotMatching.Add($matchedItem) | Out-Null
        } 
        if ($item.last_name -ne $matchedItem.last_name) {
            $transferedPartiallyLastNameNotMatching.Add($matchedItem) | Out-Null
        } 
        if ($item.encrypted_password -ne $matchedItem.encrypted_password) {
            $transferedPartiallyPasswordNotMatching.Add($matchedItem) | Out-Null
        } 
        if ($item.first_name -eq $matchedItem.first_name -and $item.last_name -eq $matchedItem.last_name -and $item.encrypted_password -eq $matchedItem.encrypted_password) {
            $fullyTransfered.Add($matchedItem) | Out-Null
        }
    }
}

$didntTransferOver | Export-Csv -Path ./didntTransferOver.csv -NoTypeInformation
$transferedPartiallyFirstNameNotMatching | Export-Csv -Path ./transferedPartiallyFirstNameNotMatching.csv -NoTypeInformation
$transferedPartiallyLastNameNotMatching | Export-Csv -Path ./transferedPartiallyLastNameNotMatching.csv -NoTypeInformation
$transferedPartiallyPasswordNotMatching | Export-Csv -Path ./transferedPartiallyPasswordNotMatching.csv -NoTypeInformation
$fullyTransfered | Export-Csv -Path ./fullyTransfered.csv -NoTypeInformation
```
Using ArrayList as opposed to standard arrays in PowerShell can indeed have an increased memory and performance impact due to the extra functionality provided by the ArrayList class. However, this increased usage is generally minimal and is unlikely to be noticeable unless you are working with a very large dataset or within a very resource-constrained environment.

If your script frequently changes the size of the collection (such as adding or removing elements), the overhead of creating a new array each time can add up and be less efficient than using an ArrayList.
