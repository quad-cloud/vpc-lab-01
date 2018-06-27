## CloudFormation Example


#### Step - 1 [Create a VPC]
Let us first create a VPC with a CIDR range of 10.0.0.16
Create a file **vpc-template.json** and copy the following content
```json
{
  "Parameters": {
    "CIDRRange": {
      "Description": "VPCCIDR Range (will be a /16 block)",
      "Type": "String",
      "Default": "10.251.0.0",
      "AllowedValues": ["10.250.0.0","10.251.0.0"]
    }
  },
  "Resources": {
    "VPCBase": {
      "Type": "AWS::EC2::VPC",
      "Properties": {
        "CidrBlock": { "Fn::Join" : ["", [{ "Ref" : "CIDRRange" }, "/16"]] },
        "EnableDnsSupport": "True",
        "EnableDnsHostnames": "True",
        "Tags": [
          { "Key": "Name", "Value":    { "Fn::Join" : ["", [{ "Ref" : "AWS::StackName" }, "-VPC"]] } }
        ]
      }
    }
  },
    "Outputs": {
    "VPCID" : { "Value" : { "Ref" : "VPCBase" } }
  }
}
```

From the aws-shell, execute the following command
```bash
aws> cloudformation create-stack --stack-name quad-vpc-b3 --template-body file://vpc-template-01.json
```
#### Step - 2 [Add a Subnet]
Now, let us create a Subnet in this VPC.
Add the following snippet to the **"Resources"** section of the template
```json
"PublicNetAZ1": {
  "Type": "AWS::EC2::Subnet",
  "Properties": {
    "AvailabilityZone": { "Fn::Select": [ "0", { "Fn::GetAZs": { "Ref": "AWS::Region" } } ] },
    "CidrBlock": { "Fn::FindInMap" : [ "VPCRanges", { "Ref": "CIDRRange"}, "PublicSubnetAZ1"] },
    "MapPublicIpOnLaunch": "True",
    "Tags": [
      { "Key": "Name", "Value": { "Fn::Join" : ["", [{ "Ref" : "AWS::StackName" }, "-PublicAZ1"]] } }
    ],
    "VpcId": { "Ref": "VPCBase" }
  }
```

Also add the following **"Mappings"** section to the template
```json
"Mappings": {
  "VPCRanges": {
    "10.250.0.0": {
      "PublicSubnetAZ1"   : "10.250.0.0/22",
      "PublicSubnetAZ2"   : "10.250.4.0/22",
      "PrivateSubnetAZ1"  : "10.250.32.0/21",
      "PrivateSubnetAZ2"  : "10.250.40.0/21"
    },
    "10.251.0.0": {
      "PublicSubnetAZ1"   : "10.251.0.0/22",
      "PublicSubnetAZ2"   : "10.251.4.0/22",
      "PrivateSubnetAZ1"  : "10.251.32.0/21",
      "PrivateSubnetAZ2"  : "10.251.40.0/21"
    }
  }
}
```

And the following snippet to the **"Mappings"** section
```json
"SubnetPublicAZ1" : { "Value" : { "Ref" : "PublicNetAZ1"} }
```

Update the existing stack to create the Subnet in the VPC
```bash
aws> cloudformation update-stack --stack-name quad-vpc-b3 --template-body file://vpc-template-02.json
```

#### Step - 3 [Add 3 more Subnets]
Add the following snippet to the **"Resources"** section of the template
```json
"PublicNetAZ2": {
  "Type": "AWS::EC2::Subnet",
  "Properties": {
    "AvailabilityZone": { "Fn::Select": [ "1", { "Fn::GetAZs": { "Ref": "AWS::Region" } } ] },
    "CidrBlock": { "Fn::FindInMap" : [ "VPCRanges", { "Ref": "CIDRRange"},  "PublicSubnetAZ2" ] },
    "MapPublicIpOnLaunch": "True",
    "Tags": [
      { "Key": "Name", "Value": { "Fn::Join" : ["", [{ "Ref" : "AWS::StackName" }, "-PublicAZ2"]] } }
    ],
    "VpcId": { "Ref": "VPCBase" }
  }
},
"PrivateNetAZ1": {
  "Type": "AWS::EC2::Subnet",
  "Properties": {
    "AvailabilityZone": { "Fn::Select": [ "0", { "Fn::GetAZs": { "Ref": "AWS::Region" } } ] },
    "CidrBlock": { "Fn::FindInMap" : [ "VPCRanges", { "Ref": "CIDRRange"},  "PrivateSubnetAZ1" ] },
    "MapPublicIpOnLaunch": "False",
    "Tags": [
      { "Key": "Name", "Value": { "Fn::Join" : ["", [{ "Ref" : "AWS::StackName" }, "-PrivateAZ1"]] } },
      { "Key": "Network", "Value": "private" }
    ],
    "VpcId": { "Ref": "VPCBase" }
  }
},
"PrivateNetAZ2": {
  "Type": "AWS::EC2::Subnet",
  "Properties": {
    "AvailabilityZone": { "Fn::Select": [ "1", { "Fn::GetAZs": { "Ref": "AWS::Region" } } ] },
    "CidrBlock": { "Fn::FindInMap" : [ "VPCRanges", { "Ref": "CIDRRange"},  "PrivateSubnetAZ2" ] },
    "MapPublicIpOnLaunch": "False",
    "Tags": [
      { "Key": "Name", "Value": { "Fn::Join" : ["", [{ "Ref" : "AWS::StackName" }, "-PrivateAZ2"]] } },
      { "Key": "Network", "Value": "private" }
    ],
    "VpcId": { "Ref": "VPCBase" }
  }
}
```

Add the following snippet to the **"Outputs"** section of the template
```json
"SubnetPublicAZ2" : { "Value" : { "Ref" : "PublicNetAZ2"} },
"SubnetPrivateAZ1" : { "Value" : { "Ref" : "PrivateNetAZ1"} },
"SubnetPrivateAZ2" : { "Value" : { "Ref" : "PrivateNetAZ2"} }
```

Update the existing stack to create the 3 Subnets in the VPC
```bash
aws> cloudformation update-stack --stack-name quad-vpc-b3 --template-body file://vpc-template-03.json
```
