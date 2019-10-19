# Custom Rules for Rule Tile 

###### *Contribution by: johnsoncodehk*

This helps to create new custom Rule Tile with new matching options, instead of the default options: Don't Care, This and Not This for the default Rule Tile. Using this will create clickable options for each Rule in your custom Rule Tile.

### Usage

From the Assets menu, select Create/Custom Rule Tile Script. This will prompt you to create a new file with a name. After creating the file, you can edit it to add new matching options and the algorithm for testing matches.

### Examples

#### Custom Attributes

```csharp
public class MyTile1 : RuleTile {

	public string tileId;
	public bool isWalkable;

}
```

#### Custom Rules

```csharp
public class MyTile2 : RuleTile<MyTile2.Neighbor> {

	public bool isWalkable;

	public class Neighbor {
		public const int DontCare = 0;
		public const int Walkable = 1;
	}

	public override bool RuleMatch(int neighbor, TileBase other) {
        if (other is RuleOverrideTile)
            other = (other as RuleOverrideTile).m_InstanceTile;

		switch (neighbor) {
			case Neighbor.DontCare:
				return true;
			case Neighbor.Walkable:
				return (other is MyTile)
					&& (other as MyTile).isWalkable;
		}

		return true;
	}
}
```

#### Expansion Rules

```csharp
public class MyTile3 : RuleTile<MyTile3.Neighbor> {

	public bool isWalkable;

	public class Neighbor : RuleTile.TilingRule.Neighbor {
		public const int Walkable = 3; // 0, 1, 2 in 'RuleTile.TilingRule.Neighbor'
	}

	public override bool RuleMatch(int neighbor, TileBase other) {
        if (other is RuleOverrideTile)
            other = (other as RuleOverrideTile).m_InstanceTile;

		switch (neighbor) {
			case Neighbor.Walkable:
				return (other is MyTile)
					&& (other as MyTile).isWalkable;
		}

		return base.RuleMatch(neighbor, other);
	}
}
```

#### Siblings Tile by whitelist (Bad performance)

```csharp
public class MyTile4 : RuleTile<MyTile4.Neighbor> {

	public List<TileBase> sibings = new List<TileBase>();

	public override bool RuleMatch(int neighbor, TileBase other) {
        if (other is RuleOverrideTile)
            other = (other as RuleOverrideTile).m_InstanceTile;

		switch (neighbor) {
			case RuleTile.TilingRule.Neighbor.This:
				return sibings.Contains(other);
		}

		return base.RuleMatch(neighbor, other);
	}
}
```

#### Siblings Tile by enum (Good performance)

```csharp
public class MyTile5 : RuleTile<MyTile5.Neighbor> {

    public enum SibingGroup
    {
		None,
        Poles,
        Terrain,
		/* ... */
    }
    public SibingGroup sibingGroup;

	public override bool RuleMatch(int neighbor, TileBase other) {
        if (other is RuleOverrideTile)
            other = (other as RuleOverrideTile).m_InstanceTile;

		switch (neighbor) {
			case RuleTile.TilingRule.Neighbor.This:
				return (other is MyTile)
					&& (other as MyTile).siblingGroup != SibingGroup.None;
					&& (other as MyTile).siblingGroup == this.siblingGroup;
		}

		return base.RuleMatch(neighbor, other);
	}
}
```

### Custom neighbor rules GUI

```csharp
[CustomEditor(typeof(MyTile3))]
public class MyTile3Editor : RuleTileEditor {

	public override void RuleOnGUI(Rect rect, int arrowIndex, int neighbor)
	{
		switch (neighbor)
		{
			case MyTile3.Neighbor.Walkable:
				GUI.DrawTexture(rect, /* ... */);
				break;
		}

		base.RuleOnGUI(rect, arrowIndex, neighbor);
	}
}
```
