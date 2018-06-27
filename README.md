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

#### Step - 4 [Internet Gateway]
Provision an Internet Gateway and associate it with the VPC
Add the following snippet to the **"Resources"** section of the template
```json
"IGWBase" : {
  "Type" : "AWS::EC2::InternetGateway",
  "Properties" : {
    "Tags" : [
      { "Key": "Name", "Value": { "Fn::Join" : ["", [{ "Ref" : "AWS::StackName" }, "-IGW"]] } }
    ]
  }
},
"VGAIGWBase" : {
  "Type" : "AWS::EC2::VPCGatewayAttachment",
  "Properties" : {
    "InternetGatewayId" : { "Ref" : "IGWBase" },
    "VpcId" : { "Ref" : "VPCBase" }
  }
}
```

Update the existing stack
```bash
aws> cloudformation update-stack --stack-name quad-vpc-b3 --template-body file://vpc-template-04.json
```

#### Step - 5 [Route Tables and Subnet Association]
Provision Route Tables and associate them with Subnets
Add the following snippet to the **"Resources"** section of the template
```json
"RouteTablePublic" : {
  "Type" : "AWS::EC2::RouteTable",
  "Properties" : {
    "VpcId" : { "Ref" : "VPCBase" },
    "Tags" : [
      { "Key": "Name", "Value": { "Fn::Join" : ["", [{ "Ref" : "AWS::StackName" }, "-PublicRT"]] } }
    ]
  }
},
"RouteTablePrivateAZ1" : {
  "Type" : "AWS::EC2::RouteTable",
  "Properties" : {
    "VpcId" : { "Ref" : "VPCBase" },
    "Tags" : [
      { "Key": "Name", "Value": { "Fn::Join" : ["", [{ "Ref" : "AWS::StackName" }, "-PrivateAZ1RT"]] } }
    ]
  }
},
"RouteTablePrivateAZ2" : {
  "Type" : "AWS::EC2::RouteTable",
  "Properties" : {
    "VpcId" : { "Ref" : "VPCBase" },
    "Tags" : [
      { "Key": "Name", "Value": { "Fn::Join" : ["", [{ "Ref" : "AWS::StackName" }, "-PrivateAZ2RT"]] } }
    ]
  }
},
"RoutePublicDefault" : {
  "DependsOn": [ "VGAIGWBase" ],
  "Type" : "AWS::EC2::Route",
  "Properties" : {
    "RouteTableId" : { "Ref" : "RouteTablePublic" },
    "DestinationCidrBlock" : "0.0.0.0/0",
    "GatewayId" : { "Ref" : "IGWBase" }
  }
},
"RouteAssociationPublicAZ1Default" : {
  "Type" : "AWS::EC2::SubnetRouteTableAssociation",
  "Properties" : {
    "SubnetId" : { "Ref" : "PublicNetAZ1"},
    "RouteTableId" : { "Ref" : "RouteTablePublic" }
  }
},
"RouteAssociationPublicAZ2Default" : {
  "Type" : "AWS::EC2::SubnetRouteTableAssociation",
  "Properties" : {
    "SubnetId" : { "Ref" : "PublicNetAZ2"},
    "RouteTableId" : { "Ref" : "RouteTablePublic" }
  }
},
"RouteAssociationPrivateAZ1Default" : {
  "Type" : "AWS::EC2::SubnetRouteTableAssociation",
  "Properties" : {
    "SubnetId" : { "Ref" : "PrivateNetAZ1"},
    "RouteTableId" : { "Ref" : "RouteTablePrivateAZ1" }
  }
},
"RouteAssociationPrivateAZ2Default" : {
  "Type" : "AWS::EC2::SubnetRouteTableAssociation",
  "Properties" : {
    "SubnetId" : { "Ref" : "PrivateNetAZ2"},
    "RouteTableId" : { "Ref" : "RouteTablePrivateAZ2" }
  }
}
```

Update the existing stack
```bash
aws> cloudformation update-stack --stack-name quad-vpc-b3 --template-body file://vpc-template-05.json
```

#### Step - 6 [Route Tables and Subnet Association]
Provision NAT Gateway, add route for NAT Gateway.
Add the following snippet to the **"Resources"** section of the template

```json
"NATAZ1" : {
  "Type" : "AWS::EC2::NatGateway",
  "DependsOn" : "VGAIGWBase",
  "Properties" : {
    "AllocationId" : { "Fn::GetAtt" : ["EIPNATAZ1", "AllocationId"]},
    "SubnetId" : { "Ref" : "PublicNetAZ1"}
  }
},
"EIPNATAZ1" : {
  "Type" : "AWS::EC2::EIP",
  "Properties" : {
    "Domain" : "vpc"
  }
},
"NATAZ1Route" : {
  "Type" : "AWS::EC2::Route",
  "Properties" : {
    "RouteTableId" : { "Ref" : "RouteTablePrivateAZ1" },
    "DestinationCidrBlock" : "0.0.0.0/0",
    "NatGatewayId" : { "Ref" : "NATAZ1" }
  }
},
"NATAZ2" : {
  "Type" : "AWS::EC2::NatGateway",
  "DependsOn" : "VGAIGWBase",
  "Properties" : {
    "AllocationId" : { "Fn::GetAtt" : ["EIPNATAZ2", "AllocationId"]},
    "SubnetId" : { "Ref" : "PublicNetAZ2"}
  }
},
"EIPNATAZ2" : {
  "Type" : "AWS::EC2::EIP",
  "Properties" : {
    "Domain" : "vpc"
  }
},
"NATAZ2Route" : {
  "Type" : "AWS::EC2::Route",
  "Properties" : {
    "RouteTableId" : { "Ref" : "RouteTablePrivateAZ2" },
    "DestinationCidrBlock" : "0.0.0.0/0",
    "NatGatewayId" : { "Ref" : "NATAZ2" }
  }
}
```

Add the following snippet to the **"Outputs"** section of the template
```json
"ElasticIP1" : { "Value": { "Ref" : "EIPNATAZ1" } },
"ElasticIP2" : { "Value": { "Ref" : "EIPNATAZ2" } },
```

Update the existing stack
```bash
aws> cloudformation update-stack --stack-name quad-vpc-b3 --template-body file://vpc-template-06.json
```